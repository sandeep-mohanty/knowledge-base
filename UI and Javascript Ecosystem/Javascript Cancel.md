# Why You Should Cancel More in JavaScript (Seriously)

Today we are going to talk about an interesting JavaScript API that pulled me out of a very common issue with web applications. I was working on a web applications which had three fetch requests running at once, and the user clicked away before any of them finished. Guess what? All three still ran to completion. Network wasted. CPU wasted.

That was the day I realized canceling async tasks isn’t optional, it’s essential. Have you been in that situation? When users click away, triggering network calls that pile up like a digital traffic jam. I needed a way to stop those async operations.

Let’s see how this cool JavaScript feature gives you control over async, making your apps faster and your users happier.

---

### Here’s How it Works
The JavaScript API we are going to talk about is called **AbortController**. It’s a cool feature for your async operations.

`AbortController` lets you cancel these operations on demand. It creates a controller object with a **signal** you can pass into things like `fetch`, `setTimeout`, or even custom async functions. When you call `abort()`, every task tied to that signal stops.



Here’s a simple example:
```javascript
const controller = new AbortController();
const { signal } = controller;

fetch('/api/data', { signal })
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Fetch canceled');
    }
  });

// Later…
controller.abort(); // Cancels the fetch
```
The signal ties the `fetch` to the controller, and `abort()` cuts it off. Instead of keeping network requests after the user moves on, you can abort it instantly.

---

### Real-World Example: Canceling a Search
Ever built a search input that fires requests on every keystroke? Without cancellation, you risk race conditions: the response from “a” might arrive after the response from “abc,” overwriting correct results with stale ones.

With `AbortController`, you can cancel outdated requests as new ones come in.

```javascript
let controller = null;

function search(query) {
  // Cancel any previous request
  if (controller) controller.abort();
  controller = new AbortController();
  fetch(`https://api.example.com/search?q=${query}`, { signal: controller.signal })
    .then(response => response.json())
    .then(data => console.log('Results:', data))
    .catch(err => {
      if (err.name === 'AbortError') return; // Expected cancellation
      console.error('Error:', err);
    });
}
// Simulate user typing
search('cat');
search('cats'); // Cancels 'cat' request
```
See that? Only the latest request survives. Your server is not bombarded anymore.

---

### Beyond Fetch: Timers and Intervals
`AbortController` isn’t just for fetch. You can wire it into almost anything async. For example, you can use it to manage `setTimeout` or `setInterval` too.

Here’s a timer that stops if the user bails:
```javascript
const controller = new AbortController();
const { signal } = controller;

function delay(ms, signal) {
  return new Promise((resolve, reject) => {
    const id = setTimeout(resolve, ms);
    signal.addEventListener("abort", () => {
      clearTimeout(id);
      reject(new Error("Timeout canceled"));
    });
  });
}

delay(5000, signal).then(() => console.log("Done")).catch(console.error);

// Cancel before 5s
controller.abort();
```
This way, timeouts won’t keep hanging around when they’re no longer needed.

---

### Why This Matters
Using `AbortController` can declutter your app. Fewer wasted requests mean faster responses. Canceled timers mean no surprise side effects. Your app feels polished, and your users stay happy.

* **Performance:** Kill requests that no one cares about. Save bandwidth and CPU.
* **UX:** Users don’t see flickering, stale data, or frozen screens.
* **Control:** You decide what lives and what dies in your app’s async operations.

---

### The Catch
Older browsers (think pre-2018) don’t support `AbortController`. Some APIs, like older WebSocket implementations, don’t play nice with signals. Make sure you test thoroughly for your use case.

### Final Takeaway
I used to accept wasted requests as part of the web application. Now I can’t imagine writing an interactive app without `AbortController`.