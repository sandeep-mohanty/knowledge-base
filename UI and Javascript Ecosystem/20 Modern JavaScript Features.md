# 20 Modern JavaScript Features So Clean They Feel Illegal to Write

There is a specific feeling that hits when you discover a JavaScript feature you did not know existed. It is closer to the feeling of finding a shortcut through a city you thought you already knew by heart. The streets were always there. You just had not walked down them yet.

Modern JavaScript is full of those streets. Features that make async logic read like a story. Operators that collapse five lines of defensive code into one. Syntax that communicates intent so clearly it almost feels like cheating. This is a tour of 20 of them.

---

### 1. let and const: The End of var
`var` is function-scoped, not block-scoped, which means it leaks out of `if` blocks and `for` loops in ways that cause silent bugs.

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Prints: 3, 3, 3 (Shared across all iterations)
```

`let` fixes this by being block-scoped. Each loop iteration gets its own `i`.

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Prints: 0, 1, 2
```

`const` is for values you do not intend to reassign. Use `const` by default, `let` only when reassigning, and never use `var`.

---

### 2. Arrow Functions: Concise Syntax and Lexical this
Arrow functions look cleaner and solve the oldest `this` trap in JavaScript.

```javascript
const doubled = numbers.map(n => n * 2);
```

Regular functions create their own `this` context. Arrow functions inherit `this` from the surrounding scope, making them perfect for callbacks inside classes or constructors.



---

### 3. Template Literals: Strings That Do Not Fight You
Template literals use backticks and `${}` for interpolation, natively supporting multi-line strings and complex expressions.

```javascript
const html = `
  <div class="card">
    <h2>${title}</h2>
    <p>Total: ${(price * quantity).toFixed(2)}</p>
  </div>
`;
```

---

### 4. Destructuring: Unpacking Data Elegantly
Extract values from arrays and objects without repetitive property access.

```javascript
// Object destructuring with renaming
const { name: userName, address: { city } } = user;

// Array destructuring
const [primary, secondary, ...remaining] = ['red', 'green', 'blue', 'yellow'];
```

---

### 5. Default Parameters
Set fallback values right in the parameter list instead of checking `undefined` manually.

```javascript
function createUser(name, role = 'viewer') {
  // role is 'viewer' if not provided or undefined
}
```

---

### 6. Spread and Rest (...)
* **Spread:** Expands an iterable into individual elements (e.g., merging arrays or cloning objects).
* **Rest:** Collects multiple elements into an array (e.g., handling variable function arguments).

---

### 7. Classes
JavaScript classes are syntactic sugar over the prototype system, making inheritance far more readable.

```javascript
class Dog extends Animal {
  speak() {
    return `${this.name} barks.`;
  }
  static square(n) { return n * n; } // Belongs to class, not instance
}
```

---

### 8. Modules (import / export)
Modules let you split code into files with explicit interfaces.

```javascript
// Exporting
export function add(a, b) { return a + b; }
export default class User { ... }

// Importing
import User, { add as addNumbers } from './utils.js';
```

---

### 9. Promises
A Promise represents a value available in the future: pending, fulfilled, or rejected.

```javascript
fetchUser(1)
  .then(user => console.log(user))
  .catch(err => console.error(err))
  .finally(() => console.log('Done'));
```
`Promise.all` runs them in parallel, while `Promise.allSettled` waits for all outcomes regardless of rejection.

---

### 10. Symbol
A `Symbol` is a unique, immutable primitive guaranteed to be different from every other Symbol, useful for private-ish object keys.

---

### 11. async / await
Built on Promises, it lets async code read like synchronous code.

```javascript
async function loadUserPosts(id) {
  try {
    const user = await fetchUser(id);
    const posts = await fetchPosts(user.id);
    render(posts);
  } catch (err) {
    handleError(err);
  }
}
```

---

### 12. Object.entries() and Object.values()
* `Object.values()`: Returns an array of values.
* `Object.entries()`: Returns an array of `[key, value]` pairs, perfect for loops.

---

### 13. Optional Chaining (?.)
Short-circuits property access if the reference is `null` or `undefined`, returning `undefined` instead of throwing a `TypeError`.

```javascript
const city = user?.address?.city;
user?.save?.(); // Safe method call
```

---

### 14. Nullish Coalescing (??)
Returns the right-hand side only when the left side is `null` or `undefined`. Unlike `||`, it keeps `0`, `''`, and `false`.

```javascript
const timeout = config.timeout ?? 5000; // If timeout is 0, it stays 0
```

---

### 15. Array.flat() and Array.flatMap()
* `flat()`: Flattens nested arrays to a specified depth.
* `flatMap()`: Maps then flattens one level deep; more efficient than `.map().flat()`.

---

### 16. Logical Assignment (&&=, ||=, ??=)
Shorthand operators combining logic with assignment.

```javascript
user.role ||= 'viewer'; // Sets if falsy
user.preferences ??= {}; // Lazy initialization if null/undefined
```

---

### 17. globalThis
One universal reference to the global object across all environments (Browser `window`, Node `global`, Worker `self`).

---

### 18. String.replaceAll()
Replaces all occurrences of a substring without needing a global regex.

```javascript
'foo foo'.replaceAll('foo', 'bar'); // 'bar bar'
```

---

### 19. Array.at()
Accepts negative indices to access elements from the end of an array or string.

```javascript
const last = arr.at(-1); // Much cleaner than arr[arr.length - 1]
```

---

### 20. Error Cause
Chain errors together to preserve the full debugging history.

```javascript
throw new Error('Failed to load config', { cause: err });
```



---

### The Honest Takeaway
If you are still writing JavaScript the way it was written in 2012, you are carrying unnecessary weight. These features are the language telling you there is a cleaner way. Start with **optional chaining** and **nullish coalescing**, then move to **async / await** and **destructuring**. 
