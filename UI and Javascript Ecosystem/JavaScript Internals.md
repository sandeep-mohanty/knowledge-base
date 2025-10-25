# JavaScript Internals: Understanding Runtime Optimization and How to Write Performant Code

From a straightforward browser scripting language, JavaScript has morphed into an ultra-flexible and versatile technology that powers everything from dynamic front-end UIs and back-end services (Node.js) to automation scripts and IoT devices (with libraries like Johnny-Five). Yet, that flexibility introduces a lot of complexity in writing efficient, performant code. 

Fortunately, [JavaScript](https://dzone.com/articles/the-role-of-javascript-in-front-end-and-back-end-d) engines working to execute your code employ a number of optimization strategies during runtime to improve performance. For these optimizations to be most effective, though, you as the developer must understand how the engines practically work and adopt coding practices that kowtow to their internal processes.

This article will, for the most part, stick to the basics of how the engines work and what kinds of practical, everyday coding strategies you can utilize to just get more oomph out of your engine.

## **The Journey of JavaScript Code**

Your JavaScript code executes after it passes through three primary stages in the JavaScript runtime:

### **AST Generation (Abstract Syntax Tree)**

The code is parsed and translated into an [Abstract Syntax Tree](https://dzone.com/articles/javascript-parser-to-create-abstract-syntax-treeas) (AST). This structure stands between the source code and the machine language. The engine processes the AST, so the format is crucial.

### **Bytecode Compilation**

The AST is then compiled into bytecode, a lower-level intermediate code that is closer to machine code, but still independent of the platform. This bytecode is run by the JavaScript engine.

### **JIT Optimization**

When the bytecode runs, the JavaScript engine continually optimizes the code using Just-In-Time (JIT) compilation. The JIT compiler collects data about the runtime (like types and usage patterns) and creates efficient machine code tailored to the environment where the code is running. These phases are critical to comprehend if you're going to write fast JavaScript. It's not enough just to write code that works; you need to structure it in a way that lets the engine optimize it effectively. Then it will run faster and perform better.

## **The Importance of JIT Compilation**

JavaScript is a language with dynamic typing, which means types are determined at runtime rather than at compile-time. This allows for flexibility in the language, but it poses certain challenges for JavaScript engines. When the engine compiles JavaScript to machine code, it has very little information about the kinds of variables or the types of function arguments that were used in the code. With so little information, the engine cannot generate highly optimized machine code at the outset.

This is where the [JIT compiler](https://dzone.com/articles/just-in-time-jit-compilation-advantages-disadvanta) comes in and does its work. JIT compilers are capable of watching the code execute and gathering information at runtime about what kinds of variables and types are being used, and they then use this information to optimize the code as it runs. By optimizing the code based on actual usage, the JIT compiler can produce highly efficient machine code for the hot paths in your code — i.e., the frequently executed parts. But not every part of the code is eligible for this kind of optimization.

## **Writing Optimization-Friendly Code**

One of the most crucial aspects of JavaScript programming that can be efficiently handled by the JIT compiler is maintaining consistency in your code, especially where types are concerned. When you create functions that take arguments of different types or forms, it makes it hard for the JIT compiler to guess what you really meant and thus what kinds of optimizations it can apply.

### Type Consistency: A Key to Performance

To showcase the significance of type consistency, let's evaluate a basic instance. Imagine you possess the following function:

```javascript
function get_x(obj) {
  return obj.x;
}
```

At first glance, this function seems to work in a very straightforward manner — it just returns the x property of an object. However, JavaScript engines have to evaluate the location of the x property during execution. This is because x could be either a direct property of the object we're passing in or something that our engine has to check for along the prototype chain. The engine also has to check to see if the property exists at all, which augments any overhead costs tied to the function's execution.

Now consider that we're calling this function with objects of varying shape:

```javascript
get_x({x: 3, y: 4});
get_x({x: 5, z: 6});
```

Here, the `get_x` function gets objects with different property sets (x, y, z) that make it tough for the engine to optimize. Each time the function is called, the engine must check to determine where the x property exists and whether it’s valid.

However, if you standardize the structure of the objects being passed, even if some properties are undefined, the engine can optimize the function:

```javascript
get_x({x: 3, y: 4, z: undefined});
get_x({x: 5, y: undefined, z: 6});
```

Now, the function is consistently receiving objects that maintain the same shape. This consistency allows the engine to optimize property access more effectively. When the engine accesses properties, it can now predict with a greater degree of certainty that certain properties will be present. For example, it knows that an "x" property will always be present — even if the engine also knows that the "x" property is undefined a significant amount of the time.

### Measuring Optimization in Action

Modern JavaScript engines offer a way to monitor the optimization of your code. You can, for instance, use Node.js to observe the optimization workings of the engine on your code.

```javascript
function get_x(obj) {
  return obj.x + 4;
}

// Run a loop to trigger optimization
for (let i = 1; i <= 1000000; i++) {
  get_x({x: 3, y: 4});
}
```

Executing this code with Node.js using the `--trace-deopt` and `--trace-opt` flags enables you to glimpse how the engine optimizes the function.

```bash
node --trace-deopt --trace-opt index.js
```

In the output, you might see something like this:

```bash
[marking 0x063d930d45d9 <JSFunction get_x (sfi = 0x27245c029f41)> for optimization to TURBOFAN, ConcurrencyMode::kConcurrent, reason: small function]
```

This message shows that the `get_x` function has been set aside for optimization and will be compiled with the very fast TURBOFAN compiler.

### **Understanding Deoptimization**

Understanding deoptimization — when the JIT compiler gives up an optimized code path — is just as important as understanding optimization. Deoptimization occurs when the engine's assumptions about the types or structures of data no longer hold true.

For instance, if you change the function to manage another object shape:

```javascript
function get_x(obj) {
  return obj.x + 4;
}

// Initial consistent calls
for (let i = 1; i <= 1000000; i++) {
  get_x({x: 3, y: 4});
}

// Breaking optimization with different object shape
get_x({z: 4});
```

The output may contain a message about deoptimization:

```
[bailout (kind: deopt-eager, reason: wrong map): begin. deoptimizing 0x1239489435c1 <JSFunction get_x...>]
```

This means that the engine had to give up its optimized code path because the new object shape wasn't what it was expecting. The function `get_x` is no longer working on objects that have a reliable structure, and so the engine has reverted to a version of the function that is not as fast.

### **Maintaining Optimization With Consistent Object Shapes**

To prevent deoptimization and to help the engine maintain optimizations, it's crucial to keep object shapes consistent. For instance:

```javascript
function get_x(obj) {
  return obj.x + 4;
}

// All calls use the same object shape
for (let i = 1; i <= 1000000; i++) {
  get_x({x: 3, y: 4, z: undefined});
}

get_x({x: undefined, y: undefined, z: 4});
```

Here, the `get_x` function always gets objects with the same shape. This lets the engine maintain its optimizations since there's no need to deoptimize when an object's shape is consistent.

## **Best Practices for Optimizable Code**

To ensure that the JavaScript code is maximally efficient and optimized by the JIT compiler, follow these best practices.

1.  **Consistent object shapes**: Make sure that the function receives the same set of properties from the object, even if some of the properties are undefined.
2.  **Type stability**: Maintain consistency in types for function parameters and return values. Don't switch between primitive types (e.g., string, number) in the same function.
3.  **Direct property access**: Optimizing direct property access (like obj.x) is easier than optimizing dynamic property access (like obj\["x"\]), so it's best to avoid dynamic lookups when you can.
4.  **Focus on hot code paths**: Focus your optimization activities on the parts of your code that are run the most. Profiling tools can help you direct your optimization efforts to the areas of your code that will benefit the most from those efforts.
5.  **Minimize type variability**: Refrain from dynamically altering the type or shape of objects and arrays. The engine can make better assumptions and optimizations when structures remain static.

## **Conclusion**

To write performant JavaScript, you need more than just a clean syntax; you need a solid understanding of how the JavaScript engine optimizes and executes your code. This means maintaining type consistency, using consistent object shapes, and knowing how the JIT compiler works. 

Right now, it is essential to remember that if you start trying to make your code perform well at every conceivable point, you will end up making it perform poorly at some points that matter and wasting your time in the process. The point, then, is not to write optimized code but rather to write code that is simple to understand and reason about, hitting the few places where it needs to be performant in a straight path from start to finish.