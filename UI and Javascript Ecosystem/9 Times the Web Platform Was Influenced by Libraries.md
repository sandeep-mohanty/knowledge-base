# 9 Times the Web Platform Was Influenced by Libraries | Jad Joubran

The web platform didn't invent most of its best APIs. It caught up to them.

Libraries did the R&D work in production. They got tested by thousands of developers across thousands of codebases, which is the kind of feedback you can't simulate. They got bug reports. They iterated. And, the patterns that survived eventually became part of the platform itself.

If you've been writing JavaScript for a while, this article will be a walk down memory lane. If you're relatively newer to the web, welcome. Here's the lineage of the APIs you use every day.

## 1\. `querySelector` and `querySelectorAll`

Picking elements out of the DOM with a CSS selector is something we now take for granted. It wasn't always built in.

Before this, you had to compose the result yourself: `getElementById`, `getElementsByClassName`, `getElementsByTagName`, then walk into `.children` and filter from there. Anything beyond a single class or tag turned into a small DOM-traversal routine.

`dojo.query` shipped this idea in Dojo. Then jQuery's `$()` (powered by the _Sizzle_ engine) made it the default mental model for an entire generation of web developers. Browsers eventually shipped their own version:

```
const button = document.querySelector(".buy-now");
const allButtons = document.querySelectorAll(".buy-now");

console.log(button.textContent);

allButtons.forEach((btn) => {
  console.log(btn.textContent);
});
```

`querySelector` returns the first match (or `null`). `querySelectorAll` returns a `NodeList` of all matches.

**Note:** Chrome and Firefox DevTools expose `$` and `$$` as shortcuts for these in the console. They're handy for poking around, but worth knowing the limits:

-   They only work at the top level of the DevTools console. They don't work in the page's JavaScript.
-   They don't work inside callbacks.
-   If the page itself defines `$` (e.g. a page using jQuery), the page's version wins.

So great for quick experiments, not something you'd rely on.

## 2\. Declarative UI: `popovertarget` and `command`

These two attributes are interesting because they let you replace JavaScript with HTML.

Their job is specific: declaratively wire a button to a target element. Click the button, the target gets shown or hidden. No `addEventListener`, no manual `.show()` / `.hide()` calls.

Ten years ago, that wiring looked like this:

```
const button = document.querySelector(".open-modal");
const modal = document.querySelector("#my-modal");

button.addEventListener("click", () => {
  modal.classList.add("is-open");
});
```

Bootstrap saw this pattern repeating everywhere and replaced the wiring with attributes:

```
<button data-toggle="modal" data-target="#my-modal">Open</button>
```

Bootstrap's jQuery plugin read the `data-*` attributes and did the show/hide for you. You wrote no JavaScript for the wiring itself.

The platform absorbed that exact idea, without the library:

```
<button popovertarget="my-popover">Open</button>

<div id="my-popover" popover>
  <p>Hello from a popover.</p>
  <button popovertarget="my-popover" popovertargetaction="hide">Close</button>
</div>
```

`command` extends the same pattern to other built-in elements (like `<dialog>`) by naming the action explicitly:

```
<button commandfor="my-dialog" command="show-modal">Open dialog</button>

<dialog id="my-dialog">
  <p>Hello from a dialog.</p>
  <button commandfor="my-dialog" command="close">Close</button>
</dialog>
```

**As a bonus**, the elements these attributes typically target — the `popover` attribute and `<dialog>` — bring their own built-in behaviors. Focus management, `Escape` to close, and (for popovers) light-dismiss all come built in. That's behavior that used to be hand-rolled in every modal implementation, and getting the accessibility right was hard.

## 3\. `classList`

Before `classList`, you manipulated the `className` property as a space-separated string. Adding a class meant splitting, checking, joining. Removing one meant filtering. It was tedious, and people got it wrong.

jQuery's `.addClass()`, `.removeClass()`, `.toggleClass()`, and `.hasClass()` were the answer. They became so ubiquitous that the platform shipped its own version under the `classList` property:

```
const button = document.querySelector(".buy-now");

button.classList.add("is-loading");
button.classList.remove("is-disabled");
button.classList.toggle("is-active");
button.classList.contains("is-loading"); // boolean
button.classList.replace("is-loading", "is-success"); // Not available in jQuery
```

A small detail worth pointing out: you pass the class name without the leading `.`. That dot was a common mistake when people first switched from CSS selectors to `classList`.

## 4\. Utility methods on strings and arrays

A whole category of small helper methods came straight from utility libraries: Underscore, Lodash, MooTools, Prototype.js. They were so common in every codebase that standardizing them was overdue.

A non-exhaustive list:

-   `String.prototype.startsWith`
-   `String.prototype.endsWith`
-   `String.prototype.includes`
-   `String.prototype.repeat`
-   `String.prototype.padStart`
-   `String.prototype.padEnd`
-   `Array.prototype.includes`
-   `Array.prototype.flat` (from `_.flatten`)
-   `Array.prototype.flatMap`

Every one of these used to be a line of `npm install` (or a `<script>` tag) away. Now they're just there.

A small piece of web history: `Array.prototype.flat` was originally proposed as `flatten`, but MooTools had monkey-patched `Array.prototype.flatten` with different semantics. Adding a native `flatten` would have broken sites still loading MooTools, so the proposal was renamed, in what's now known as [SmooshGate](https://developer.chrome.com/blog/smooshgate).

## 5\. `structuredClone`

Deep cloning an object in [JavaScript](https://learnjavascript.online/?utm_source=jadjoubran.io/blog) used to mean reaching for `_.cloneDeep` from Lodash, or doing the famous `JSON.parse(JSON.stringify(obj))` trick that breaks the moment you have a `Date`, a `Map`, or a circular reference.

Here's the twist worth highlighting: the deep-clone algorithm was _already_ in the platform. Any time you called `postMessage` or wrote an object to IndexedDB, the browser quietly used it to copy your data. We just had no way to call it on its own.

`structuredClone` is the platform finally exposing what it had been doing internally for years:

```
const original = {
  name: "Jad",
  createdAt: new Date("2026-01-01"),
  tags: new Map([["role", "instructor"]]),
  nested: { items: [1, 2, 3] },
};

const copy = structuredClone(original);

copy.nested.items.push(4);

console.log(original.nested.items); // [1, 2, 3] (unchanged)
console.log(copy.nested.items); // [1, 2, 3, 4]
console.log(copy.createdAt instanceof Date); // true
console.log(copy.tags instanceof Map); // true
```

`Date`, `Map`, `Set`, `RegExp`, and typed arrays all survive the clone with their original types intact. No library required.

## 6\. Promises

Async work in JavaScript used to mean reaching for one of several competing libraries: Dojo `Deferred`, jQuery `Deferred`, Q, Bluebird. The ecosystem eventually converged on a shared spec called Promises/A+, and the platform adopted it.

Bluebird is worth a small note. It was so much faster than the early native implementations that many codebases kept using it for years after Promises shipped natively. That's not a common outcome — the library outperforming the platform — but it shows how seriously the userland work was taken.

Today, Promises and `async`/`await` are the language's default model for async:

```
async function loadUser(id) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error("Failed to load user");
  }
  return response.json();
}

const user = await loadUser(42);
console.log(user.name);
```

## 7\. ES Modules

Splitting code across files is fundamental, but JavaScript shipped without a module system. The community filled the gap.

Node.js used CommonJS (`require` / `module.exports`). Browsers used AMD via RequireJS (`define([...], function () { ... })`). Two userland systems, years of bundler tooling, and finally the platform shipped its own:

```
// math.js
export function add(a, b) {
  return a + b;
}

export const PI = 3.14159;
```

```
// app.js
import { add, PI } from "./math.js";

console.log(add(2, 3)); // 5
console.log(PI); // 3.14159
```

The `import` / `export` syntax is its own thing. It's not what CommonJS used, and it's not what AMD used. But the _concept_ of modules came from those libraries proving the pattern was needed.

## 8\. Temporal

JavaScript's `Date` object has been broken for thirty years. Mutable, zero-indexed months, no real time-zone support. Libraries grew up around it. Moment.js was the dominant one, and it catalogued every `Date` pain point in existence.

Here's the part that matters: the `Temporal` proposal was championed by Moment.js maintainer Maggie Johnson-Pint (then at Microsoft), with engineers from Bloomberg and Igalia driving much of the multi-year specification and implementation effort.

```
const birthday = Temporal.PlainDate.from("2026-06-27");
const reminder = birthday.subtract({ days: 7 });

console.log(reminder.toString()); // 2026-06-20
console.log(birthday.toString()); // 2026-06-27 (unchanged)
```

For a deep dive on `Temporal` itself, see my earlier post: [Working with Dates in JavaScript Using the Temporal API](/blog/javascript-temporal-api) and the [Temporal API Cheatsheet](https://learnjavascript.online/temporal.html).

## 9\. `Element.closest()`

Walking up the DOM to find the nearest matching ancestor used to mean a hand-rolled `while` loop, or jQuery's `.closest()`. It's now built in.

The canonical use case is event delegation:

```
document.addEventListener("click", (event) => {
  const button = event.target.closest("button.action");
  if (!button) return; // click wasn't on (or inside) a matching button

  const action = button.dataset.action;
  console.log("action:", action);
});
```

You bind one listener at the document level, and `closest()` figures out which button (if any) was actually clicked, even if the click landed on an icon or text node _inside_ the button.

## So what's next?

Not every platform feature starts in a library — plenty come straight from browser engineers and standards bodies. But this particular path, where userland probes, iterates, and the survivors get standardized, is one of the healthier ones. It's a feedback loop more than a competition.

Imagine a world where you had to `npm install` a library just to call `document.querySelector`. That was the world. We forget how much of what feels native today started in someone's `lib/` folder.

The platform absorbing library patterns isn't libraries "losing." It's them succeeding so completely that they become unnecessary.

So here's the fun question: which of today's libraries are tomorrow's standards? What patterns are quietly being battle-tested in userland _right now_ that will land in the platform in a few years? And considering that browsers are moving faster than ever — features sometimes reach Baseline Newly Available within the same year they ship — that timeline might be even shorter than it used to be.