# 9 Rules That’ll Help You Write Clean Code

You write code every day. It runs. Tests pass. Tickets close. But six months later, you open that same file and think:
“Who wrote this?”

It was you. And that’s the problem.
Clean code isn’t about being clever. It’s about being kind to the next person who reads your work, and most of the time, that person is future you. These 9 rules won’t just clean up your codebase. They’ll change how you think about writing software altogether.

Let’s get into it.

---

### 1. You Ain’t Gonna Need It (YAGNI)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*jUuEvDuCO6a3laf6TUS3Vg.png)

This one saves more time than almost any other rule on this list.
YAGNI comes from Extreme Programming, and the idea is simple: don’t write code for a future you can’t predict. Developers have a habit of thinking ahead. “What if we need this later?” And so they build a feature, a toggle, an abstraction, for a requirement that never actually comes.

That unused code still has to be read. Still has to be maintained. Still has to be understood by every new developer who joins the team.
Build what you need today. Design for change, yes. But don’t write code for imaginary tomorrows.

---

### 2. Single Responsibility Principle (SRP)

A quick test for you: Can you describe what a function or class does without using the word “and”?
If not, it’s doing too much.

Robert C. Martin, also known as Uncle Bob, defined SRP as part of the famous SOLID principles back in 2000. His exact words:
> “A class should have only one reason to change.”

Not one thing, one reason to change. That’s a subtle but important distinction.
A login module should not change because the checkout flow changed. A function that fetches data should not also format it for display. Keep responsibilities focused, and your code becomes dramatically easier to test, maintain, and understand.

---

### 3. Don’t Repeat Yourself (DRY)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*bBm_oFWcH8GFi1JtRomQkA.png)

Andy Hunt and Dave Thomas coined this one in *The Pragmatic Programmer*, and it might be the most quoted rule in software development:
> “Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.”

In plain terms, write it once and use it everywhere.
Bugs multiply when logic is duplicated. If you fix it in one place but miss the other three, you have just shipped a problem. **DRY** solves that by making sure changes happen in one place.

One caveat worth knowing: **DRY** can be overdone. Forcing abstractions where they don’t belong creates code that’s technically non-repetitive but completely unreadable. The “Rule of Three” is a good gut check: duplicate once, abstract on the third occurrence.

---

### 4. Fail Fast (FF)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*p4LvTvgLvy6yCoz6OTncvQ.png)

This one doesn’t get discussed enough, and it should.
The "fail fast" principle is straightforward: detect errors as early as possible and stop immediately. Don’t let bad data silently travel deep into your system before it causes a problem. Surface the error right where it happens.

Eric Raymond put it perfectly in *The Art of Unix Programming*:
> “Repair what you can, but when you must fail, fail noisily and as soon as possible.”

The benefit is real. When something breaks close to where the bad input entered, debugging takes minutes instead of hours.

---

### 5. Keep It Simple, Stupid (KISS)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/0*E6UI7mXUWY9a-22D.png)

Fun fact: KISS didn’t come from software. The U.S. Navy first noted it in 1960 and attributed it to Kelly Johnson, lead engineer at Lockheed Skunk Works. He challenged his team to design aircraft that an average mechanic in a field, under combat conditions, could repair with only basic tools.

That’s the standard. Can someone maintain the performance under pressure with limited context?
In software, KISS means resisting the urge to write clever code. That brilliant one-liner you’re proud of? The next developer will spend 20 minutes decoding it. Readable code beats clever code every single time. Simple systems are easier to debug, easier to extend, and easier to hand off.

---

### 6. Test Driven Development (TDD)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*6-nYgn0dqXI4-zc9eGgzDA.png)

Kent Beck developed TDD in the late 1990s as part of Extreme Programming, though he famously called it a *rediscovery* rather than an invention. The cycle is known as **Red, Green, Refactor**:
1. Write a failing test (**Red**)
2. Write just enough code to make it pass (**Green**)
3. Clean up the code without changing behaviour (**Refactor**)

What most people miss is that TDD isn’t really about testing. It’s about design. Writing the test first forces you to think about your API before your implementation. It makes you define what the code should do before figuring out how to do it. The tests are just the safety net that lets you refactor with confidence later.

---

### 7. Separation of Concerns (SOC)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/0*It2xa6lJ2Cen4pS8.png)

Edsger Dijkstra introduced this concept way back in 1974, and it’s so foundational that two of the five SOLID principles are direct derivations of it.

The core idea: don’t write your program as one big block. Break it into distinct sections where each one handles a single, specific concern. UI logic lives separately from business logic. Database access lives separately from both.

A practical way to think about it: if changing your database requires touching your UI code, your concerns are not separated. When they are properly isolated, you can modify one part of the system without setting the rest on fire.

---

### 8. Document Your Code (DYC)

[Image showing self-explanatory code with 'why' comments vs messy code with 'what' comments]

Here’s the thing about documentation that nobody says clearly enough: **don’t comment on *what* the code does, comment on *why* it does it.**

Good code explains itself. A well-named function, a clear variable, a logical structure—these tell you what is happening. What they can’t always tell you is why a specific decision was made, why a particular edge case is handled the way it is, or why an obvious-looking approach was deliberately avoided.

That’s what comments are for. Write for the developer who joins your team in a year, the one with no context, no Slack history, and a deadline. Because that developer might also be you.

---

### 9. Boy Scout Rule (BSR)

![](https://miro.medium.com/v2/resize:fit:720/format:webp/0*NLdFUsnX1dWBVH_6.png)

Uncle Bob adapted this from an actual Boy Scouts principle:
> “Always leave the campground cleaner than you found it.”

Applied to code, it means this: every time you touch a file, leave it slightly better than you found it. Rename a confusing variable. Remove a dead comment. Break up a function that’s grown too large. You don’t have to refactor the whole codebase. Just make one small improvement per commit.

The compounding effect is real. If five developers each make one small improvement per commit, pushing a few commits a day, the codebase gets meaningfully better every single week without a single “refactoring sprint” ever being scheduled.

Clean code isn’t a one-time project. It’s a daily habit.

---

### The Bigger Picture

These 9 rules share one idea at their core:
**Write code for humans, not just computers.**

Computers will run almost anything you throw at them. It’s your teammates, your future self, and everyone who maintains this system after you who need the code to be clear, intentional, and honest.

Start with one rule. Apply it consistently. Then pick another.
That’s how good codebases are built.