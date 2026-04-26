# You Already Use Functors. They’re Called Arrays.
**The functional concepts you’re avoiding are hiding in plain JavaScript.**

You ever hear someone say “functor” and immediately feel like they were trying to make you feel stupid?

Same.

I used to flinch at functional jargon. Monads, functors, endofunctors; it all sounded like a cruel inside joke from a CS professor who never had to debug production code at 2 AM. I figured it wasn’t for me. Not for real devs.

But then one day, I read what a functor actually is. And I laughed. Out loud.

Because it turns out… I’d been using them for years.

Let me guess: you’ve written something like this?

```javascript
const nums = [1, 2, 3];
const doubled = nums.map(n => n * 2);
```

Boom. That’s it. That’s a functor in action.

---

### So what is a functor, really?

Here’s the academic definition, distilled:

> **A functor is a structure you can map over.**



It holds a value (or values), and you can apply a function to those values without changing the structure itself. In JavaScript, the humble array fits the bill perfectly:

* It’s a **container** for values.
* You can call **.map(fn)** on it.
* The result is **still an array**, just with transformed values.

So `[1, 2, 3]` becomes `[2, 4, 6]`, and the array-ness stays intact.

In functional terms: **arrays preserve the context while transforming the content.**

---

### Still sounds academic? Let’s go deeper.

Imagine you wrote a library like this:

```javascript
const Box = (value) => ({
  map: (fn) => Box(fn(value)),
  fold: (fn) => fn(value),
});
```

You can use it like this:

```javascript
Box(3)
  .map(x => x + 1)
  .map(x => x * 2)
  .fold(console.log); // 8
```

That’s a functor too. Just a custom one. It wraps a value, lets you **.map** over it, and returns a new wrapped value.

Sound familiar?

It should. Arrays do the same, just with multiple values inside. The difference is: arrays are built-in, so you don’t even think about it.

---

### Why does this matter?

Because once you understand that **.map()** isn’t just a convenience, but a manifestation of a pattern, **you start to see** the hidden architecture of clean code:

* **You stop fearing abstractions.** When someone says “functor”, you’ll just hear “oh, so it maps over a thing.”
* **You spot bugs faster.** A proper functor guarantees structure. **.map()** doesn’t flatten, mutate, or bail so transformations are safe and predictable.
* **You write more composable code.** Mapping over things becomes your go-to instead of branching and checking and mutating and praying.

---

### Almost-Functors Are Everywhere (And That’s the Problem)

Most things in JavaScript are almost functors… but not quite.

* **Objects?** No native **.map()**.
* **Promises?** Kinda. **.then()** behaves like a map, but it unwraps.
* **Nullables?** You have to write checks… unless you build your own **Maybe**.

That’s why people build their own functors. To give any value this map-friendly behavior, and **start reasoning with a uniform mental model.**

Once you unlock that, you’re halfway to understanding monads. (But don’t run away just yet.)

---

### Final thoughts

JavaScript’s been whispering functional ideas to you all along. Quietly. Through its quirks, its syntax, its everyday methods.

**map** is a door. And once you walk through, the rest of functional programming doesn’t seem so intimidating anymore.

It just feels like naming things properly.

Like calling a functor… a functor.

Even when it’s just an array.