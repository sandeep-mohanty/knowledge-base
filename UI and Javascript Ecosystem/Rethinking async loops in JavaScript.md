# Rethinking async loops in JavaScript

Using `await` in loops seems intuitive until your code silently stalls or runs slower than expected. If you’ve ever wondered why your API calls run one-by-one instead of all at once, or why `map()` and `await` don’t mix the way you’d expect, grab a chair. Let’s chat.

## The problem: awaiting in a `for` loop

Suppose you’re fetching a list of users one-by-one:

```javascript
const users = [1, 2, 3];

for (const id of users) {
  const user = await fetchUser(id);
  console.log(user);
}
```

This works, but it runs sequentially: `fetchUser(2)` doesn’t start until `fetchUser(1)` finishes. That’s fine if order matters, but it’s inefficient for independent network calls.

## Don’t `await` inside `map()` unless you mean to

A common point of confusion is using `await` inside `map()` without handling the resulting promises:

```javascript
const users = [1, 2, 3];

const results = users.map(async id => {
  const user = await fetchUser(id);
  return user;
});

console.log(results); // [Promise, Promise, Promise] – NOT actual user data
```

This works in terms of syntax and behavior (it returns an array of promises), but not in the way many expect. It doesn’t wait for the promises to resolve.

To run the calls in parallel and get the final results:

```javascript
const results = await Promise.all(users.map(id => fetchUser(id)));
```

Now all the requests run **in parallel**, and `results` contains the actual fetched users.

## `Promise.all()` fails fast, even if just one call breaks

When using `Promise.all()`, a **single rejection** causes the entire operation to fail:

```javascript
const results = await Promise.all(
  users.map(id => fetchUser(id)) // fetchUser(2) might throw
);
```

If `fetchUser(2)` throws an error (e.g., 404 or network error), the entire `Promise.all` call will reject, and **none** of the results will be returned (including successful ones).

## Safer alternatives

### Use `Promise.allSettled()`

```javascript
const results = await Promise.allSettled(
  users.map(id => fetchUser(id))
);

results.forEach(result => {
  if (result.status === 'fulfilled') {
    console.log('✅ User:', result.value);
  } else {
    console.warn('❌ Error:', result.reason);
  }
});
```

Use this when you want to process **all results**, even if some fail.

### Handle errors inside the mapping function

```javascript
const results = await Promise.all(
  users.map(async id => {
    try {
      return await fetchUser(id);
    } catch (err) {
      console.error(`Failed to fetch user ${id}`, err);
      return { id, name: 'Unknown User' }; // fallback value
    }
  })
);
```

This also prevents **unhandled promise rejections**, which can trigger warnings or crash your process in stricter environments like Node.js with `--unhandled-rejections=strict`.

## Modern solutions

### Use `for...of` + `await` (sequential execution)

Use when the next operation depends on the result of the previous one, or when API rate limits require it:

```javascript
for (const id of users) {
  const user = await fetchUser(id);
  console.log(user);
}
```

Or if you’re not in an `async` function context:

```javascript
(async () => {
  for (const id of users) {
    const user = await fetchUser(id);
    console.log(user);
  }
})();
```

-   Maintains order
-   Useful for rate-limiting or batching
-   Slower for independent requests

### Use `Promise.all` + `map()` (parallel execution)

Use when operations are independent and can be performed simultaneously:

```javascript
const usersData = await Promise.all(users.map(id => fetchUser(id)));
```

-   Much faster for network-heavy or CPU-independent tasks
-   One rejection causes the whole batch to fail (unless handled)

Use `Promise.allSettled()` or inline `try/catch` for safer batch execution.

### Throttled parallelism (controlled concurrency)

When you need speed but must respect API limits, use a throttling utility like [`p-limit`](https://www.npmjs.com/package/p-limit):

```javascript
import pLimit from 'p-limit';

const limit = pLimit(2); // Run 2 fetches at a time
const limitedFetches = users.map(id => limit(() => fetchUser(id)));

const results = await Promise.all(limitedFetches);
```

-   Balance between concurrency and control
-   Prevents overloading external services
-   Adds dependency

## Concurrency levels

Goal

Pattern

Concurrency

Keep order, run one-by-one

`for...of` + `await`

1

Run all at once, no order

`Promise.all()` + `map()`

∞ (unbounded) ✅

Limit concurrency

`p-limit`, `PromisePool`, etc.

N (custom-defined)

## Last tip: never use `await` in `forEach()`

This is a common trap:

```javascript
users.forEach(async id => {
  const user = await fetchUser(id);
  console.log(user); // ❌ Not awaited
});
```

The loop doesn’t wait for your `async` function. These fetches run in the background with no guarantee on completion timing or order.

Instead, use:

-   `for...of` + `await` for sequential logic
-   `Promise.all()` + `map()` for parallel logic

## Quick recap

JavaScript’s async model is powerful, but using `await` inside loops requires intention. Here’s the key: structure your async logic **based on your needs**.

-   Order → `for...of`
-   Speed → `Promise.all()`
-   Safety → `allSettled()` / `try-catch`
-   Balance → `p-limit`, etc.

With the right pattern, you can write faster, safer, more predictable asynchronous code.
