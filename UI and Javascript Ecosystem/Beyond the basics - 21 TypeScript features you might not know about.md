# Beyond the basics: 21 TypeScript features you might not know about

## [](#introduction)Introduction

At [Lingo.dev](https://lingo.dev/?utm_source=devto&utm_medium=organic&utm_content=lingodotdev/beyond-the-basics-21-typescript-features-you-might-not-know-about-1dbn), I write a lot of TypeScript code. I'm definitely not a wizard, but I do try to play with features that go beyond the basic types.

This post describes a number of features (and when you might want to use them) to help you expand your knowledge beyond the absolute fundamentals.

## [](#1-readonly-arrays-tuples-and-raw-as-const-endraw-assertions)1\. Readonly arrays, tuples, and `as const` assertions

By default, arrays and objects are mutable, and TypeScript widens literal values to their general types. This makes it harder for TypeScript to help you catch bugs and provide accurate autocomplete.  

```typescript
const colors = ["red", "green", "blue"];
// Type: string[] - could be any strings
colors.push("yellow"); // Allowed, might not be what you want

type Color = (typeof colors)[number]; // string (too general!)
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Use `as const` to make everything readonly and preserve literal types, or use `readonly` for specific arrays.  

```typescript
const colors = ["red", "green", "blue"] as const;
// Type: readonly ["red", "green", "blue"]
colors.push("yellow"); // ✗ Error: can't modify readonly array

type Color = (typeof colors)[number]; // "red" | "green" | "blue" ✓

// Or for function parameters:
function display(items: readonly string[]) {
  items.push("x"); // ✗ Error: can't modify
  items.forEach(console.log); // ✓ OK: reading is fine
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Configuration or constant data that shouldn't change
-   Preventing accidental mutations
-   Preserving literal types for better type inference
-   Function parameters that shouldn't be modified

**Learn more:** [TypeScript Docs: ReadonlyArray](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#readonlyarray)

## [](#2-raw-keyof-typeof-endraw-for-objectasconst-enums)2\. `keyof typeof` for object-as-const enums

TypeScript enums have some quirks and generate JavaScript code. Sometimes you just want to define constants in an object and derive types from them.

### [](#solution)Solution

Combine `as const` (to lock in literal values), `typeof` (to get the object's type), and `keyof` (to get the union of keys or values).  

```typescript
// Define your constants as a plain object
const STATUS = {
  PENDING: "pending",
  APPROVED: "approved",
  REJECTED: "rejected",
} as const; // Lock in the literal values

// Get a union of the values
type Status = (typeof STATUS)[keyof typeof STATUS];
// "pending" | "approved" | "rejected"

function setStatus(status: Status) {
  // TypeScript validates and autocompletes!
}

setStatus(STATUS.APPROVED); // ✓
setStatus("pending"); // ✓
setStatus("invalid"); // ✗ Error
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Alternative to enums with better JavaScript output
-   Creating const-based configuration with derived types
-   When you want both runtime values and compile-time types

**Learn more:** [TypeScript Docs: typeof types](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)

## [](#3-labeled-tuple-elements)3\. Labeled tuple elements

Tuples like `[number, number, boolean]` work, but it's not obvious what each position means. Is it `[width, height, visible]` or `[x, y, enabled]`?

### [](#solution)Solution

Give tuple positions meaningful names that show up in your editor's autocomplete and error messages.  

```typescript
// Before: unclear what each number means
type Range = [number, number, boolean?];

// After: self-documenting
type Range = [start: number, end: number, inclusive?: boolean];

function createRange([start, end, inclusive = false]: Range) {
  // Your editor will show you the parameter names!
  return { start, end, inclusive };
}

createRange([1, 10, true]); // Clear what each argument means
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Function parameters that are tuple-based
-   Return values with multiple related pieces of data
-   Any tuple where the meaning of positions isn't obvious

**Learn more:** [TypeScript Docs: tuple types](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)

## [](#4-indexed-access-and-element-type-extraction)4\. Indexed access and element type extraction

You have a complex type and want to refer to just one property's type, or extract what's inside an array, without repeating yourself.

### [](#solution)Solution

Use bracket notation (`Type["property"]`) to access property types, and `[number]` to get array element types.  

```typescript
type User = {
  id: number;
  profile: {
    name: string;
    emails: string[];
  };
};

// Access nested property types
type ProfileType = User["profile"]; // { name: string; emails: string[] }
type NameType = User["profile"]["name"]; // string

// Extract array element type
type Email = User["profile"]["emails"][number]; // string
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Deriving types from existing types (DRY principle)
-   Extracting array/tuple element types
-   Working with nested structures without redefining types

**Learn more:** [TypeScript Docs: indexed access types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)

## [](#5-userdefined-type-guards-raw-arg-is-t-endraw-)5\. User-defined type guards (`arg is T`)

You write a function that checks if something is a certain type, but TypeScript doesn't understand that the check actually narrows the type.  

```typescript
function isPerson(x: unknown) {
  return typeof x === "object" && x !== null && "name" in x;
}

function greet(x: unknown) {
  if (isPerson(x)) {
    x.name; // ✗ Error: TypeScript still thinks x is 'unknown'
  }
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Use a type predicate (`arg is Type`) to tell TypeScript that your function performs a type check.  

```typescript
type Person = { name: string; age: number };

function isPerson(x: unknown): x is Person {
  return (
    typeof x === "object" &&
    x !== null &&
    "name" in x &&
    typeof (x as any).name === "string"
  );
}

function greet(x: unknown) {
  if (isPerson(x)) {
    console.log(x.name); // ✓ TypeScript knows x is Person here!
  }
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Validating data from APIs or user input
-   Type-safe validation functions
-   Discriminating between types in a union

**Learn more:** [TypeScript Docs: type predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)

## [](#6-exhaustive-checking-with-raw-never-endraw-)6\. Exhaustive checking with `never`

You have a union type (like different shapes or statuses) and a switch statement. Later, someone adds a new variant to the union but forgets to handle it in the switch. No error is thrown - it just silently doesn't work.

### [](#solution)Solution

Add a `default` case that assigns the value to a `never` type. If all cases are handled, the default is unreachable. If a case is missing, TypeScript will error because the value isn't assignable to `never`.  

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      // If all cases are handled, this is unreachable
      const _exhaustive: never = shape;
      throw new Error(`Unhandled shape: ${_exhaustive}`);
  }
}

// Later, someone adds triangle:
// type Shape = ... | { kind: "triangle"; base: number; height: number };
// ✓ TypeScript error in default case: triangle is not assignable to never!
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Switch statements over discriminated unions
-   Ensuring all union variants are handled
-   Catching bugs when types evolve over time

**Learn more:** [TypeScript Docs: exhaustiveness checking](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking)

## [](#7-typeonly-imports-and-exports-raw-import-type-endraw-raw-export-type-endraw-)7\. Type-only imports and exports (`import type`/`export type`)

Sometimes you import types from other modules, but those imports show up in your compiled JavaScript even though they're only used for type checking. This can cause circular dependencies or bundle bloat.

### [](#solution)Solution

Use `import type` to tell TypeScript: "this is only needed for type checking, erase it completely from the JavaScript."  

```typescript
// Regular import - might end up in compiled JS
import { User } from "./types";

// Type-only import - guaranteed to be removed from JS
import type { User } from "./types";

// Mixed imports
import { saveUser, type User } from "./api";
//       ^^^^^^^^^  ^^^^^^^^^^^
//       value      type-only
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Preventing circular dependency issues
-   Keeping your JavaScript bundle smaller
-   When using build tools that require explicit type-only imports (`isolatedModules`)
-   Clarifying intent (this is only for types, not runtime code)

**Learn more:** [TypeScript Docs: importing types](https://www.typescriptlang.org/docs/handbook/2/modules.html#importing-types)

## [](#8-ambient-module-declarations-for-noncode-assets)8\. Ambient module declarations for non-code assets

You import non-TypeScript files (like images, CSS, or data files), but TypeScript doesn't know what type they should have.  

```typescript
import logo from "./logo.svg"; // ✗ Error: Cannot find module
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Create ambient module declarations that tell TypeScript how to type these imports.  

```typescript
// In a .d.ts file (like global.d.ts or declarations.d.ts)
declare module "*.svg" {
  const url: string;
  export default url;
}

declare module "*.css" {
  const classes: { [key: string]: string };
  export default classes;
}

// Now these work:
import logo from "./logo.svg"; // logo: string
import styles from "./app.css"; // styles: { [key: string]: string }
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Typing imports of images, fonts, styles
-   JSON or data files not handled by your build tool
-   Any non-TypeScript asset your bundler processes

**Learn more:** [TypeScript Docs: module declaration templates](https://www.typescriptlang.org/docs/handbook/declaration-files/templates/module-d-ts.html)

## [](#9-the-raw-satisfies-endraw-operator)9\. The `satisfies` operator

Sometimes you want TypeScript to check that an object matches a type, but you _also_ want TypeScript to remember the specific values you used (not just that they're strings or numbers).  

```typescript
// Without satisfies - loses specific information
const routes: Record<string, string> = {
  home: "/",
  profile: "/users/:id",
};
// routes.profile is just 'string', not the specific "/users/:id"
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

`satisfies` checks your object against a type _without_ changing what TypeScript remembers about it.  

```typescript
const routes = {
  home: "/",
  profile: "/users/:id",
} satisfies Record<string, `/${string}`>; // Must be strings starting with "/"

// routes.profile is still the literal "/users/:id" - exact value preserved!
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Configuration objects where you want both validation AND specific value types
-   When you need autocomplete on exact values, not just the general type

**Learn more:** [TypeScript Docs: satisfies operator](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#the-satisfies-operator)

## [](#10-assertion-functions-raw-asserts-endraw-and-raw-asserts-x-is-t-endraw-)10\. Assertion functions (`asserts` and `asserts x is T`)

Sometimes you want a function that throws an error if a condition isn't met. Type guards (above) only work in `if` statements - they don't affect the code after the function call.  

```typescript
function assertNotNull(x: unknown) {
  if (x == null) throw new Error("Value is null!");
}

const data: string | null = getValue();
assertNotNull(data);
// TypeScript still thinks data might be null here
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Assertion functions use `asserts` to tell TypeScript: "if this function returns (doesn't throw), then the condition is true."  

```typescript
function assertNotNull<T>(x: T): asserts x is NonNullable<T> {
  if (x == null) throw new Error("Value is null!");
}

const data: string | null = getValue();
assertNotNull(data);
// ✓ TypeScript now knows data is definitely string here!
data.toUpperCase(); // Safe to use
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Validation functions that throw on failure
-   Enforcing runtime invariants
-   Early error checking at function boundaries

**Learn more:** [TypeScript Docs: assertion functions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#assertion-functions)

## [](#11-template-literal-types-for-string-patterns)11\. Template literal types for string patterns

Imagine you have event names like `"user:login"`, `"user:logout"`, `"post:create"`, etc. You want TypeScript to autocomplete these and catch typos, but there are too many to list manually.

### [](#solution)Solution

Template literal types let you describe string patterns using the same syntax as JavaScript template strings.  

```typescript
// Generate all combinations automatically
type EventName = `${"user" | "post"}:${"create" | "delete"}`;
// Result: "user:create" | "user:delete" | "post:create" | "post:delete"

function trackEvent(event: EventName) {
  // TypeScript will autocomplete and validate the event names!
}

trackEvent("user:create"); // ✓ OK
trackEvent("user:update"); // ✗ Error - not a valid combination
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   API routes or event names that follow a pattern
-   CSS class names with prefixes/suffixes
-   Any structured string format (like database table names, file paths)

**Learn more:** [TypeScript Docs: template literal types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)

## [](#12-distributive-conditional-types)12\. Distributive conditional types

You want to filter or transform a union type (like `string | number | null`) by applying logic to each member.

### [](#solution)Solution

Conditional types automatically distribute over unions when the checked type is "naked" (not wrapped in another type).  

```typescript
// Remove null and undefined from a union
type NonNullish<T> = T extends null | undefined ? never : T;

// This distributes: checks each member separately
type Clean = NonNullish<string | number | null>;
// string | number (null was filtered out)

// Extract only function types
type FunctionsOnly<T> = T extends (...args: any[]) => any ? T : never;
type Fns = FunctionsOnly<string | ((x: number) => void) | boolean>;
// (x: number) => void
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Filtering union types
-   Building utility types like `Exclude`, `Extract`
-   Transforming each member of a union differently

**Learn more:** [TypeScript Docs: distributive conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)

## [](#13-raw-infer-endraw-to-capture-types-inside-conditionals)13\. `infer` to capture types inside conditionals

You need to extract a piece of a complex type (like "what type does this function return?" or "what's inside this array?").

### [](#solution)Solution

Use `infer` to create a type variable that captures part of the type you're examining.  

```typescript
// Extract the return type of a function
type ReturnType<F> = F extends (...args: any[]) => infer R ? R : never;

type MyFunc = (x: number) => string;
type Result = ReturnType<MyFunc>; // string

// Extract array element type
type ElementType<T> = T extends (infer E)[] ? E : never;

type Numbers = ElementType<number[]>; // number
type Mixed = ElementType<(string | boolean)[]>; // string | boolean
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Extracting parameter or return types from functions
-   Getting element types from arrays or tuples
-   Parsing types inside template literals or complex structures

**Learn more:** [TypeScript Docs: inferring within conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types)

## [](#14-mapped-type-modifiers-raw-readonly-endraw-raw-endraw-etc)14\. Mapped type modifiers (`+readonly`, `-?`, etc.)

Sometimes you need to take an existing type and make all properties required (remove `?`), or make everything mutable (remove `readonly`). Manually rewriting each property is tedious.

### [](#solution)Solution

Mapped types can add (`+`) or remove (`-`) the `readonly` and optional (`?`) modifiers.  

```typescript
// Remove readonly from all properties
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// Remove optional (?) from all properties
type Required<T> = {
  [K in keyof T]-?: T[K];
};

type Config = Readonly<{ port?: number; host?: string }>;
type EditableConfig = Mutable<Required<Config>>; // { port: number; host: string }
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Creating editable versions of readonly types
-   Making all properties required for validation functions
-   Building utility types that transform property modifiers

**Learn more:** [TypeScript Docs: mapping modifiers](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers)

## [](#15-key-remapping-in-mapped-types-raw-as-endraw-)15\. Key remapping in mapped types (`as`)

You want to transform an object type by changing property names (like removing a prefix, or filtering out certain properties), but mapped types normally keep the same keys.

### [](#solution)Solution

Use `as` inside a mapped type to transform keys as you iterate over them.  

```typescript
// Remove private properties (those starting with _)
type RemovePrivate<T> = {
  [K in keyof T as K extends `_${string}` ? never : K]: T[K];
};

type WithPrivate = { name: string; _secret: number };
type Public = RemovePrivate<WithPrivate>; // { name: string }

// Add a prefix to all keys
type Prefixed<T> = {
  [K in keyof T as `app_${string & K}`]: T[K];
};

type Original = { id: number; name: string };
type Result = Prefixed<Original>; // { app_id: number; app_name: string }
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Creating public versions of types by filtering out internal properties
-   Converting between naming conventions (camelCase to snake\_case)
-   Filtering object types based on key patterns

**Learn more:** [TypeScript Docs: key remapping in mapped types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-in-mapped-types)

## [](#16-const-type-parameters)16\. Const type parameters

When you pass an array to a generic function, TypeScript normally "widens" it to a general array type, losing information about the specific values.  

```typescript
function identity<T>(value: T) {
  return value;
}

const pair = identity([1, 2]); // Type is number[], not [1, 2]
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Add `const` before the type parameter to tell TypeScript: "keep this as specific as possible."  

```typescript
function identity<const T>(value: T) {
  return value;
}

const pair = identity([1, 2]); // Type is [1, 2] - exact tuple preserved!
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Functions that should preserve exact array/tuple structures
-   Builder functions where you want to track literal values through transformations

**Learn more:** [TypeScript Docs: const type parameters](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html#const-type-parameters)

## [](#17-variadic-tuple-types-and-spreads)17\. Variadic tuple types and spreads

How do you type a function that needs to accept different numbers of arguments while keeping track of each argument's type?

### [](#solution)Solution

Variadic tuples let you work with lists of types that can grow or shrink. Think of `...` as "spread this list of types here."  

```typescript
// Type that adds an element to the end of a tuple
type Push<T extends unknown[], U> = [...T, U];

type Result = Push<[string, number], boolean>; // [string, number, boolean]

// Real example: Typing a function wrapper
function logged<Args extends unknown[], Return>(
  fn: (...args: Args) => Return,
): (...args: Args) => Return {
  return (...args) => {
    console.log("Calling with:", args);
    return fn(...args);
  };
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Wrapping functions while preserving their exact argument types
-   Creating type-safe function composition utilities
-   Building tuple manipulation types

**Learn more:** [TypeScript Docs: tuple types](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)

## [](#18-the-raw-this-endraw-parameter-in-functions)18\. The `this` parameter in functions

When a function uses `this`, TypeScript doesn't know what type `this` should have. This causes problems with methods, callbacks, and event handlers.  

```typescript
function setName(name: string) {
  this.name = name; // ✗ Error: 'this' has type 'any'
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Add an explicit `this` parameter (doesn't count as a real parameter at runtime) to type what `this` should be.  

```typescript
interface Model {
  name: string;
  setName(this: Model, newName: string): void;
}

const model: Model = {
  name: "Initial",
  setName(this: Model, newName: string) {
    this.name = newName; // ✓ TypeScript knows what 'this' is!
  },
};

model.setName("Updated"); // Works
const fn = model.setName;
fn("Test"); // ✗ Error: 'this' context is wrong
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Methods that rely on `this`
-   Event handler callbacks
-   Functions designed to be called with `.call()` or `.apply()`

**Learn more:** [TypeScript Docs: this parameters](https://www.typescriptlang.org/docs/handbook/2/functions.html#this-parameters)

## [](#19-raw-unique-symbol-endraw-for-nominallike-typing)19\. `unique symbol` for nominal-like typing

TypeScript uses "structural typing" - two types with the same structure are considered the same. Sometimes you want types that are structurally identical but _logically_ different (like UserID vs ProductID, both strings).  

```typescript
type UserId = string;
type ProductId = string;

function getUser(id: UserId) {
  /* ... */
}

const productId: ProductId = "prod-123";
getUser(productId); // ✗ We want this to be an error, but it's not!
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Use `unique symbol` to create a "brand" that makes types incompatible even if their structure is identical.  

```typescript
declare const USER_ID: unique symbol;
type UserId = string & { [USER_ID]: true };

declare const PRODUCT_ID: unique symbol;
type ProductId = string & { [PRODUCT_ID]: true };

function getUser(id: UserId) {
  /* ... */
}

const productId = "prod-123" as ProductId;
getUser(productId); // ✓ Now this IS an error!
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Preventing mixing up IDs of different types (user IDs, order IDs, etc.)
-   Creating "nominal" types that can't be accidentally substituted
-   Type-safe keys for registries or dependency injection

**Learn more:** [TypeScript Docs: unique symbol](https://www.typescriptlang.org/docs/handbook/symbols.html#unique-symbol)

## [](#20-module-augmentation-and-declaration-merging)20\. Module augmentation and declaration merging

You're using a third-party library and need to add properties to its types (like adding custom configuration options), but you can't edit the library's code.

### [](#solution)Solution

Use module augmentation to add to existing interfaces or modules from the outside.  

```typescript
// In your own .d.ts file
declare module "express" {
  // Add to Express's Request interface
  interface Request {
    user?: { id: string; name: string };
  }
}

// Now TypeScript knows about req.user in your Express handlers!
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Extending library types with custom properties
-   Adding types for library features that aren't fully typed
-   Plugin systems where you're registering new capabilities

**Learn more:** [TypeScript Docs: module augmentation](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation)

## [](#21-constructor-signatures-and-abstract-newable-types)21\. Constructor signatures and abstract "newable" types

You want to write a function that accepts a _class_ (not an instance) and creates instances of it. How do you type "something that can be constructed with `new`"?  

```typescript
function createInstance(SomeClass: ???) {
  return new SomeClass();
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#solution)Solution

Use a constructor signature: `new (...args: any[]) => T` to describe something that can be constructed.  

```typescript
// Type describing a constructor
type Constructor<T = unknown, Args extends unknown[] = any[]> = new (
  ...args: Args
) => T;

function createInstance<T>(Ctor: Constructor<T>): T {
  return new Ctor();
}

class User {
  name = "Unknown";
}

const user = createInstance(User); // user: User ✓

// More complex: factory with specific constructor arguments
function createPair<T>(
  Ctor: Constructor<T, [string, number]>,
  name: string,
  age: number,
): T {
  return new Ctor(name, age);
}
```

Enter fullscreen mode Exit fullscreen mode

### [](#when-to-use-it)When to use it

-   Dependency injection frameworks
-   Factory functions that create instances
-   Generic code that works with classes as values
-   Testing utilities that mock constructors
