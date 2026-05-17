# My 7 Killer Self-Review Secrets as a Senior Developer
**How I catch real bugs before code review, without fancy tools or extra time.**


The seven moves below are how I catch my own bugs before someone else does. None of them needs fancy tools or the best AI models. They take 15 minutes per pull request and save hours of back-and-forth.
---

### 1. Check Your Assumptions
Every function you write has things you believe are true. Half the time, they are not. I shipped this in a billing service:

```python
def add_credit(account_id, amount):
    account = Account.objects.get(id=account_id)
    account.balance += amount
    account.save()
    
    Ledger.objects.create(
        account_id=account_id,
        amount=amount,
        balance_after=account.balance,
    )
```

Looks fine. Read account, update balance, log the change. Tests passed. It ran for three weeks before finance noticed the ledger numbers did not match the actual balances.

What would you check here? The math? The order of operations? The database query?

I assumed two things: Nobody else changes the balance between my read and my write. And that the balance update and the ledger entry happen together. Both were wrong. Two requests for the same account could overlap. And the ledger insert was not inside the same transaction as the balance update.

I now write down my assumptions before I push. Then I read the code and ask if each one actually holds. Most bugs hide in the gap between what you assumed and what the code around you guarantees.

---

### 2. Imagine Two Users at Once
Code is usually written so that one user runs it on a quiet day. Real production has thousands of users at the same time. A teammate shipped this to stop duplicate webhook events:

```python
def handle_webhook(event_id, payload):
    if Event.objects.filter(event_id=event_id).exists():
        return
    
    process(payload)
    Event.objects.create(event_id=event_id)
```

Stripe sometimes sends the same webhook twice within milliseconds. Both requests checked `exists()` at the same time, both got `False`, both called `process()`. We charged a customer twice and sent two confirmation emails.



What would you do here? Add a lock? Use a queue? Check twice?

I now imagine two copies of my function running side by side, line by line. If the timing of one can mess up the other, the code is broken even when tests pass. The fix here was a unique constraint `event_id` in the database. Let the database be the referee, not your code.

---

### 3. Trace What You Just Coupled
Code reviews catch what changed. They miss what now depends on what. I added a feature flag check to a notification:

```python
def send_order_email(order):
    if flags.enabled("new_template"):
        template = NEW_TEMPLATE
    else:
        template = OLD_TEMPLATE
    
    email.send(order.user.email, template.render(order))
```

Three months later, someone deleted the flag. Someone else deleted `NEW_TEMPLATE` in a different PR. Both PRs passed review. The notification service started crashing in production because nobody connected the two changes.

What would you do here? Add tests? Add a default? Leave a comment?

I now ask one question for every dependency I add: **Who owns this, and what happens when it changes?** A feature flag is a contract with another team’s code. So is a config value. So is a column in a table you do not own. Each one is a future maintenance cost.

I list every external thing my code now depends on in the PR description. If the list is long, the change is more expensive than it looks.

---

### 4. List the Ways It Can Fail
Tests check that things work. They rarely check what happens when something else breaks. I wrote this dashboard endpoint:

```python
async def get_dashboard(user_id):
    user, orders, recommendations = await asyncio.gather(
        get_user(user_id),
        get_orders(user_id),
        get_recommendations(user_id),
    )
    return Dashboard(user, orders, recommendations)
```

The recommendation service was slow and failed about 1 in 300 times. That sounds fine. But `gather` fails the whole thing if any single call fails. So 1 in 300 users could not see their orders because their recommendations failed. Recommendations are not even important.



What would you do here? Add retries? Add a timeout? Cache the result?

I now make a small list for any function that calls more than one service. What can fail, and what should the user see when it does? For this code, recommendations are nice-to-have, so I made them fail silently to an empty list. Orders are essential, so they still fail the whole request. 

The point is to decide on purpose. Don't let the library decide for you.

---

### 5. Question Every Abstraction You Add
Junior devs write too little structure. Mid devs write too much. Senior devs write less than they want to. I caught myself building this:

```python
class NotificationStrategy(ABC):
    def should_send(self, event): ...
    def render(self, event): ...
    def deliver(self, message): ...

class EmailStrategy(NotificationStrategy): ...
class SmsStrategy(NotificationStrategy): ...
class PushStrategy(NotificationStrategy): ...
```

Clean. Extensible. Three classes are ready for new channels. Easy to add Slack later. 

We had exactly one type of notification: Order confirmations by email. The structure was solving a problem we did not have. New hires now had to read four files to understand what one email looked like.

What would you do here? Keep it for the future? Flatten it? Compromise?

I now ask three things before I add a class hierarchy or a new layer: How many real cases exist today? How likely is the third case in six months? What does it cost a reader who only needs the simple case?

If I have one or two cases and no clear third one coming, I keep it flat. I deleted the strategy pattern and replaced it with three plain functions in one file. When we added Slack months later, we extracted the structure with full knowledge of what it actually needed.

---

### 6. Walk Every Loop Slowly
Code that is fast in tests is not always fast in production. The reason is almost always inside a loop. I shipped this:

```python
def serialize_orders(orders):
    return [
        {
            "id": order.id,
            "total": order.total,
            "customer": order.customer.name,
            "items": [
                {"sku": item.sku, "name": item.product.name}
                for item in order.items.all()
            ],
        }
        for order in orders
    ]
```

In tests with 10 orders, this ran in 4 milliseconds. In production with 200 orders, it ran in 11 seconds. Why? `order.customer.name` and `item.product.name` each hit the database. Per order. Per item. The page was making thousands of queries.



What would you do here? Cache things? Paginate harder? Use a different framework?

I now read every loop and ask one question: what work happens inside, and how many times? Database calls inside a for loop. Network calls inside a map. File reads inside a comprehension.

The fix here was `prefetch_related` to load everything in 4 queries instead of thousands. The lesson is to physically count the work your loops do. Most performance problems are loops doing more work than the author noticed.

---

### 7. Ask If You Can Undo It
Some changes are easy to take back. Some are not. I treat them very differently now. I almost shipped this migration:

```python
operations = [
    migrations.RemoveField(
        model_name="order",
        name="legacy_id",
    ),
]
```

The column was unused in our code. I searched the whole codebase. Nothing referenced it. Tests passed. A reviewer asked one question: “Does anyone outside our service read this column?”

Three dashboards and a finance report queried that column daily. They ran against a replica database and never touched our application code. My grep found nothing. Dropping the column would have broken all four silently.

What would you do here? Ship it? Ask around? Mark it deprecated first?

I now sort every change by how hard it is to undo. Adding a column is easy. Changing an API response is harder. Deleting data, often impossible. For anything hard to undo, I need a real plan: a deprecation period, a flag, a backfill, or a rollback script.

The mistake juniors make is treating all changes as equally safe. The mistake mid-level devs make is trusting tests to catch the dangerous ones. Tests do not know about your reports, your mobile app users, or another team’s batch job. You have to ask.

---

### Best Practices to Build the Habit
The seven moves above only work if you actually do them. A few rules that keep me honest:

1.  **Self-review before you ask anyone else.** Open it in the browser, read your own diff, and fix what you find. Reviewers should never be your first reader.
2.  **Take a break before self-review.** Fifteen minutes away from the code does more than fifteen more minutes of staring at it. Come back cold.
3.  **Do it in the diff view.** Your editor shows the whole file, which lets you see what you meant. The diff shows only what changed, which is what reviewers see.
4.  **Read your own PR description out loud.** If it does not match what your code actually does, one of them is wrong.
5.  **Keep the PR small enough to review in 20 minutes.** Small PRs catch their own bugs because the author can actually hold the whole change in their head.
6.  **Write the test that would have caught the bug.** Not the test that proves the fix. The useful test is the one that would have failed *before* your change.
7.  **Leave the code reviewer one job.** Your goal is not a perfect PR. It is a PR where the reviewer adds one piece of context you could not have known alone.

None of these moves is clever. They are all just slowing down at the right moment. The thing senior devs do is hold two views of the code at once: What it does, and what could go wrong.

The first time you catch yourself before review, you will understand why senior devs stay quiet during reviews. They already had this conversation with themselves.