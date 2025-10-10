# Functional Programming in Python: A Comprehensive Tutorial

## Table of Contents
1. [Introduction to Functional Programming](#introduction-to-functional-programming)
2. [First-Class Functions](#first-class-functions)
3. [Pure Functions](#pure-functions)
4. [Immutability](#immutability)
5. [Lambda Functions](#lambda-functions)
6. [Higher-Order Functions](#higher-order-functions)
7. [Map, Filter, and Reduce](#map-filter-and-reduce)
8. [Function Composition](#function-composition)
9. [Recursion](#recursion)
10. [Closures](#closures)
11. [Partial Functions](#partial-functions)
12. [Function Caching](#function-caching)
13. [List Comprehensions](#list-comprehensions)
14. [Generator Expressions](#generator-expressions)
15. [The functools Module](#the-functools-module)
16. [The itertools Module](#the-itertools-module)
17. [Real-World Applications](#real-world-applications)
18. [Best Practices](#best-practices)

## Introduction to Functional Programming

Functional Programming (FP) is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing state and mutable data. Python, while not a pure functional language like Haskell or Clojure, supports many functional programming concepts.

### Key Principles of Functional Programming:

- **First-Class Functions**: Functions can be assigned to variables, passed as arguments, and returned from other functions
- **Pure Functions**: Functions that produce the same output for the same input without side effects
- **Immutability**: Data cannot be changed after it is created
- **Function Composition**: Building complex functions by combining simpler ones
- **Recursion**: Using recursive calls instead of loops for iteration

### Benefits of Functional Programming:

- **Predictability**: Pure functions make code easier to understand and test
- **Concurrency**: Immutable data and lack of side effects simplify concurrent programming
- **Modularity**: Functions as building blocks lead to more modular code
- **Testability**: Pure functions are easier to test in isolation
- **Reusability**: Higher-order functions increase code reuse

## First-Class Functions

In Python, functions are first-class citizens, meaning they can be treated like any other object.

### Assigning Functions to Variables

```python
def square(x):
    return x * x

# Assign function to a variable
f = square

# Call the function through the variable
print(f(5))  # 25
```

### Passing Functions as Arguments

```python
def apply_function(func, value):
    return func(value)

def double(x):
    return x * 2

def triple(x):
    return x * 3

print(apply_function(double, 5))  # 10
print(apply_function(triple, 5))  # 15
```

### Returning Functions from Functions

```python
def create_multiplier(factor):
    def multiplier(x):
        return x * factor
    return multiplier

double = create_multiplier(2)
triple = create_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

### Storing Functions in Data Structures

```python
def add(x, y):
    return x + y

def subtract(x, y):
    return x - y

def multiply(x, y):
    return x * y

def divide(x, y):
    return x / y

# Store functions in a dictionary
operations = {
    'add': add,
    'subtract': subtract,
    'multiply': multiply,
    'divide': divide
}

# Use functions from the dictionary
print(operations['add'](10, 5))      # 15
print(operations['subtract'](10, 5))  # 5
print(operations['multiply'](10, 5))  # 50
print(operations['divide'](10, 5))    # 2.0
```

## Pure Functions

Pure functions always produce the same output for the same input and have no side effects.

### Characteristics of Pure Functions:

1. Return value depends only on the input parameters
2. No side effects (no state changes, no I/O operations)
3. Do not modify input arguments

### Examples of Pure Functions

```python
# Pure function
def add(x, y):
    return x + y

# Pure function
def calculate_circle_area(radius):
    return 3.14159 * radius * radius
```

### Examples of Impure Functions

```python
# Impure function (has side effect - printing)
def add_and_print(x, y):
    result = x + y
    print(f"The result is {result}")  # Side effect
    return result

# Impure function (depends on external state)
counter = 0
def increment_counter():
    global counter
    counter += 1  # Side effect - modifies global state
    return counter

# Impure function (modifies input)
def add_to_list(value, my_list):
    my_list.append(value)  # Side effect - modifies input
    return my_list
```

### Converting Impure to Pure

```python
# Impure function
def add_to_list(value, my_list):
    my_list.append(value)  # Side effect - modifies input
    return my_list

# Pure version
def add_to_list_pure(value, my_list):
    return my_list + [value]  # Creates a new list instead of modifying

# Usage
original_list = [1, 2, 3]

# Impure
modified_list = add_to_list(4, original_list)
print(original_list)  # [1, 2, 3, 4] - original list was modified

# Reset for comparison
original_list = [1, 2, 3]

# Pure
new_list = add_to_list_pure(4, original_list)
print(original_list)  # [1, 2, 3] - original list unchanged
print(new_list)       # [1, 2, 3, 4] - new list created
```

## Immutability

Immutability means data cannot be changed after it is created. It's a core principle in functional programming.

### Built-in Immutable Types in Python

- Strings
- Numbers (int, float, complex)
- Tuples
- Frozensets

### Using Immutable Data Structures

```python
# Strings are immutable
name = "Alice"
# This doesn't modify the original string but creates a new one
uppercase_name = name.upper()
print(name)          # Alice (unchanged)
print(uppercase_name)  # ALICE

# Tuples are immutable
point = (2, 3)
try:
    point[0] = 5  # Will raise an error
except TypeError as e:
    print(f"Error: {e}")  # Error: 'tuple' object does not support item assignment

# Creating "updated" versions of immutable data
original_point = (2, 3)
# Create a new tuple with the modified value
new_point = (5, original_point[1])
print(original_point)  # (2, 3)
print(new_point)       # (5, 3)
```

### Simulating Immutability with Mutable Types

```python
# Using a copy to avoid modifying the original
def add_user_to_list(user, user_list):
    # Create a new list instead of modifying the original
    return user_list + [user]

users = ["Alice", "Bob"]
new_users = add_user_to_list("Charlie", users)

print(users)      # ["Alice", "Bob"]
print(new_users)  # ["Alice", "Bob", "Charlie"]

# Using frozen dataclasses (Python 3.7+)
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int

p = Point(1, 2)
try:
    p.x = 3  # Will raise an error
except Exception as e:
    print(f"Error: {e}")  # Error: cannot assign to field 'x'
```

## Lambda Functions

Lambda functions are small anonymous functions defined using the `lambda` keyword.

### Basic Syntax

```python
# Regular function
def square(x):
    return x ** 2

# Equivalent lambda function
square_lambda = lambda x: x ** 2

print(square(5))        # 25
print(square_lambda(5))  # 25
```

### Lambda with Multiple Parameters

```python
add = lambda x, y: x + y
print(add(3, 5))  # 8

# Lambda with multiple expressions (using a trick)
# This is generally not recommended for readability
complicated = lambda x, y: (
    x ** 2 + y ** 2 if x > y 
    else x * y
)
print(complicated(3, 2))  # 13 (3² + 2² = 9 + 4)
print(complicated(2, 3))  # 6 (2 * 3)
```

### Using Lambda with Built-in Functions

```python
# Sorting a list of tuples by the second element
pairs = [(1, 'one'), (3, 'three'), (2, 'two')]
sorted_pairs = sorted(pairs, key=lambda pair: pair[1])
print(sorted_pairs)  # [(1, 'one'), (3, 'three'), (2, 'two')]

# Filtering a list
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # [2, 4, 6, 8, 10]

# Mapping a list
squared_numbers = list(map(lambda x: x ** 2, numbers))
print(squared_numbers)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

### When to Use Lambda Functions

Lambda functions are best for short, simple operations, especially when used as arguments to higher-order functions. For more complex operations, named functions are usually more readable.

## Higher-Order Functions

Higher-order functions either take functions as arguments or return functions (or both).

### Functions that Accept Function Arguments

```python
def apply_twice(func, arg):
    """Apply a function to an argument, then apply it again to the result."""
    return func(func(arg))

def add_five(x):
    return x + 5

print(apply_twice(add_five, 10))  # 20 (10 + 5 + 5)
print(apply_twice(lambda x: x * 2, 3))  # 12 (3 * 2 * 2)
```

### Functions that Return Functions

```python
def make_adder(n):
    """Return a function that adds n to its argument."""
    def adder(x):
        return x + n
    return adder

add_10 = make_adder(10)
print(add_10(5))  # 15

# Using multiple returned functions
add_5 = make_adder(5)
add_7 = make_adder(7)

print(add_5(10))  # 15
print(add_7(10))  # 17
```

### Example: Function Transformer

```python
def compose(func1, func2):
    """Return a function that composes func1 and func2."""
    return lambda x: func1(func2(x))

def double(x):
    return x * 2

def increment(x):
    return x + 1

# Create a function that first increments, then doubles
double_after_increment = compose(double, increment)
print(double_after_increment(5))  # 12 (double(increment(5)) = double(6) = 12)

# Create a function that first doubles, then increments
increment_after_double = compose(increment, double)
print(increment_after_double(5))  # 11 (increment(double(5)) = increment(10) = 11)
```

## Map, Filter, and Reduce

These higher-order functions are fundamental to functional programming for data transformation.

### Map

The `map()` function applies a function to each item in an iterable.

```python
numbers = [1, 2, 3, 4, 5]

# Using map with a named function
def square(x):
    return x ** 2

squared = map(square, numbers)
print(list(squared))  # [1, 4, 9, 16, 25]

# Using map with lambda
cubed = map(lambda x: x ** 3, numbers)
print(list(cubed))  # [1, 8, 27, 64, 125]

# Map with multiple iterables
list1 = [1, 2, 3]
list2 = [4, 5, 6]
sum_lists = map(lambda x, y: x + y, list1, list2)
print(list(sum_lists))  # [5, 7, 9]
```

### Filter

The `filter()` function constructs an iterator from elements of an iterable for which a function returns true.

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Filter out odd numbers
even_numbers = filter(lambda x: x % 2 == 0, numbers)
print(list(even_numbers))  # [2, 4, 6, 8, 10]

# Filter with a named function
def is_prime(n):
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0 or n % 3 == 0:
        return False
    i = 5
    while i * i <= n:
        if n % i == 0 or n % (i + 2) == 0:
            return False
        i += 6
    return True

prime_numbers = filter(is_prime, range(1, 20))
print(list(prime_numbers))  # [2, 3, 5, 7, 11, 13, 17, 19]

# Filtering strings
names = ["Alice", "Bob", "Charlie", "David", "Eve"]
long_names = filter(lambda name: len(name) > 4, names)
print(list(long_names))  # ["Alice", "Charlie", "David"]
```

### Reduce

The `reduce()` function applies a function of two arguments cumulatively to the items of a sequence.

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# Sum all numbers
sum_result = reduce(lambda x, y: x + y, numbers)
print(sum_result)  # 15 (1 + 2 + 3 + 4 + 5)

# Product of all numbers
product_result = reduce(lambda x, y: x * y, numbers)
print(product_result)  # 120 (1 * 2 * 3 * 4 * 5)

# Find maximum value
max_value = reduce(lambda x, y: x if x > y else y, numbers)
print(max_value)  # 5

# With initial value
sum_with_initial = reduce(lambda x, y: x + y, numbers, 10)
print(sum_with_initial)  # 25 (10 + 1 + 2 + 3 + 4 + 5)

# Combining strings
words = ["Hello", "functional", "programming", "world"]
sentence = reduce(lambda x, y: x + " " + y, words)
print(sentence)  # "Hello functional programming world"
```

## Function Composition

Function composition is combining multiple functions to create a new function.

### Basic Function Composition

```python
def compose2(f, g):
    """Compose two functions: f(g(x))"""
    return lambda x: f(g(x))

# Example functions
def double(x): return x * 2
def increment(x): return x + 1
def square(x): return x ** 2

# Compose functions
double_then_increment = compose2(increment, double)
square_then_double = compose2(double, square)

print(double_then_increment(5))  # 11 (5 * 2 + 1)
print(square_then_double(3))     # 18 (3^2 * 2)
```

### Multiple Function Composition

```python
def compose(*functions):
    """Compose an arbitrary number of functions, from right to left."""
    def compose_two(f, g):
        return lambda x: f(g(x))
    
    if len(functions) == 0:
        return lambda x: x  # Identity function
    
    return functools.reduce(compose_two, functions)

# Compose three functions
f = compose(double, increment, square)
print(f(3))  # 20 (double(increment(square(3))) = double(increment(9)) = double(10) = 20)

# Compose in different order
g = compose(square, double, increment)
print(g(3))  # 64 (square(double(increment(3))) = square(double(4)) = square(8) = 64)
```

### Practical Example: Data Processing Pipeline

```python
def remove_punctuation(text):
    import string
    return text.translate(str.maketrans('', '', string.punctuation))

def lowercase(text):
    return text.lower()

def tokenize(text):
    return text.split()

def remove_stopwords(tokens):
    stopwords = {'a', 'an', 'the', 'in', 'on', 'at', 'to', 'for', 'with', 'by'}
    return [token for token in tokens if token not in stopwords]

# Create a text processing pipeline
process_text = compose(remove_stopwords, tokenize, lowercase, remove_punctuation)

text = "Hello, World! This is a test with some punctuation."
processed = process_text(text)
print(processed)  # ['hello', 'world', 'this', 'is', 'test', 'some', 'punctuation']
```

## Recursion

Recursion is a technique where a function calls itself to solve a problem. In functional programming, recursion often replaces loops.

### Basic Recursion

```python
def factorial(n):
    """Calculate n! recursively."""
    if n == 0 or n == 1:
        return 1
    else:
        return n * factorial(n - 1)

print(factorial(5))  # 120 (5 * 4 * 3 * 2 * 1)
```

### Tail Recursion

Tail recursion is a special form where the recursive call is the last operation in the function.

```python
def factorial_tail(n, accumulator=1):
    """Calculate n! using tail recursion."""
    if n == 0 or n == 1:
        return accumulator
    else:
        return factorial_tail(n - 1, n * accumulator)

print(factorial_tail(5))  # 120
```

### Recursion vs. Loops

```python
# Recursive sum
def recursive_sum(numbers):
    if not numbers:
        return 0
    return numbers[0] + recursive_sum(numbers[1:])

# Iterative sum
def iterative_sum(numbers):
    total = 0
    for num in numbers:
        total += num
    return total

numbers = [1, 2, 3, 4, 5]
print(recursive_sum(numbers))  # 15
print(iterative_sum(numbers))  # 15
```

### Handling Recursive Limits

Python has a default recursion limit to prevent stack overflow.

```python
import sys

# Check current recursion limit
print(sys.getrecursionlimit())  # Usually 1000

# Change recursion limit (use with caution)
# sys.setrecursionlimit(2000)

# Better approach: Use iteration for deep recursion
def fibonacci_recursive(n):
    if n <= 1:
        return n
    return fibonacci_recursive(n-1) + fibonacci_recursive(n-2)

def fibonacci_iterative(n):
    if n <= 1:
        return n
    a, b = 0, 1
    for _ in range(2, n+1):
        a, b = b, a + b
    return b

print(fibonacci_iterative(10))  # 55
# fibonacci_recursive(1000) would exceed recursion limit
```

## Closures

A closure is a function object that remembers values in the enclosing scope even if they are not present in memory.

### Basic Closures

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor  # 'factor' is from the enclosing scope
    return multiply

# Create closures with different factors
double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

### Closures with State

```python
def counter():
    count = 0
    
    def increment():
        nonlocal count
        count += 1
        return count
    
    return increment

# Create a counter
my_counter = counter()
print(my_counter())  # 1
print(my_counter())  # 2
print(my_counter())  # 3

# Create another independent counter
counter2 = counter()
print(counter2())  # 1
print(my_counter())  # 4 (the first counter continues from where it left off)
```

### Practical Example: Memoization with Closures

```python
def memoize(func):
    cache = {}
    
    def memoized_func(*args):
        if args in cache:
            return cache[args]
        result = func(*args)
        cache[args] = result
        return result
    
    return memoized_func

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Now fibonacci has a cached version
print(fibonacci(30))  # Calculates efficiently due to memoization
```

## Partial Functions

Partial functions allow you to fix a certain number of arguments of a function and generate a new function.

### Using functools.partial

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Create a partial function for squares
square = partial(power, exponent=2)
print(square(5))  # 25

# Create a partial function for cubes
cube = partial(power, exponent=3)
print(cube(5))  # 125

# Partial with positional arguments
def multiply(a, b, c):
    return a * b * c

double_multiply = partial(multiply, 2)
print(double_multiply(3, 4))  # 24 (2 * 3 * 4)

# Use with predefined function
from operator import add
add_five = partial(add, 5)
print(add_five(10))  # 15
```

### Creating Partial Functions Manually

```python
def partial_manually(func, *fixed_args, **fixed_kwargs):
    def new_func(*args, **kwargs):
        combined_args = fixed_args + args
        combined_kwargs = dict(fixed_kwargs)
        combined_kwargs.update(kwargs)
        return func(*combined_args, **combined_kwargs)
    return new_func

def greet(greeting, name, punctuation):
    return f"{greeting}, {name}{punctuation}"

hello_greeting = partial_manually(greet, "Hello", punctuation="!")
print(hello_greeting("Alice"))  # Hello, Alice!

formal_greeting = partial_manually(greet, "Good day", punctuation=".")
print(formal_greeting("Sir"))  # Good day, Sir.
```

## Function Caching

Function caching stores the results of expensive function calls to avoid redundant calculations.

### Using functools.lru_cache

```python
import functools
import time

# Without caching
def fibonacci_slow(n):
    if n <= 1:
        return n
    return fibonacci_slow(n-1) + fibonacci_slow(n-2)

# With caching
@functools.lru_cache(maxsize=None)
def fibonacci_fast(n):
    if n <= 1:
        return n
    return fibonacci_fast(n-1) + fibonacci_fast(n-2)

# Measure performance
def time_function(func, *args):
    start = time.time()
    result = func(*args)
    end = time.time()
    print(f"{func.__name__}({args}) = {result} (took {end - start:.6f} seconds)")
    return result

# Try with small n
time_function(fibonacci_slow, 30)  # Will take a while
time_function(fibonacci_fast, 30)  # Should be much faster

# Try again - cached version will be instant
time_function(fibonacci_fast, 30)  # Even faster due to caching
```

### Custom Caching

```python
def memoize(func):
    """Simple memoization decorator."""
    cache = {}
    
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    # Add ability to clear cache
    wrapper.clear_cache = lambda: cache.clear()
    wrapper.cache_info = lambda: f"Cache size: {len(cache)}"
    
    return wrapper

@memoize
def expensive_operation(a, b):
    print(f"Computing {a} + {b}...")
    time.sleep(1)  # Simulate expensive operation
    return a + b

print(expensive_operation(2, 3))  # Computing 2 + 3... (takes 1 second)
print(expensive_operation(2, 3))  # Returns immediately from cache
print(expensive_operation(3, 4))  # Computing 3 + 4... (takes 1 second)
print(expensive_operation.cache_info())  # Cache size: 2
expensive_operation.clear_cache()
print(expensive_operation.cache_info())  # Cache size: 0
```

## List Comprehensions

List comprehensions provide a concise way to create lists based on existing lists.

### Basic List Comprehension

```python
# Traditional way
squares = []
for i in range(10):
    squares.append(i ** 2)
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Using list comprehension
squares_comp = [i ** 2 for i in range(10)]
print(squares_comp)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### Filtering with List Comprehension

```python
# Traditional filter
even_squares = []
for i in range(10):
    if i % 2 == 0:
        even_squares.append(i ** 2)
print(even_squares)  # [0, 4, 16, 36, 64]

# Using list comprehension with filter
even_squares_comp = [i ** 2 for i in range(10) if i % 2 == 0]
print(even_squares_comp)  # [0, 4, 16, 36, 64]
```

### Nested List Comprehension

```python
# Create a 3x3 matrix
matrix = [[i + j for j in range(3)] for i in range(0, 9, 3)]
print(matrix)  # [[0, 1, 2], [3, 4, 5], [6, 7, 8]]

# Flatten a matrix
flattened = [cell for row in matrix for cell in row]
print(flattened)  # [0, 1, 2, 3, 4, 5, 6, 7, 8]

# Transpose a matrix
transposed = [[row[i] for row in matrix] for i in range(3)]
print(transposed)  # [[0, 3, 6], [1, 4, 7], [2, 5, 8]]
```

### Dictionary and Set Comprehensions

```python
# Dictionary comprehension
squares_dict = {i: i**2 for i in range(5)}
print(squares_dict)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Dictionary from two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
mapping = {k: v for k, v in zip(keys, values)}
print(mapping)  # {'a': 1, 'b': 2, 'c': 3}

# Set comprehension
square_set = {i**2 for i in range(-5, 5)}
print(square_set)  # {0, 1, 4, 9, 16, 25}
```

## Generator Expressions

Generator expressions are similar to list comprehensions but create generators instead of lists, using less memory.

### Basic Generator Expression

```python
# List comprehension (eager - creates full list in memory)
squares_list = [x**2 for x in range(10)]
print(squares_list)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Generator expression (lazy - generates values on-demand)
squares_gen = (x**2 for x in range(10))
print(squares_gen)  # <generator object <genexpr> at 0x...>

# Consume generator values
for square in squares_gen:
    print(square, end=' ')  # 0 1 4 9 16 25 36 49 64 81
print()
```

### Memory Efficiency

```python
import sys

# Compare memory usage
list_comp = [i for i in range(10000)]
gen_exp = (i for i in range(10000))

print(f"List size: {sys.getsizeof(list_comp)} bytes")
print(f"Generator size: {sys.getsizeof(gen_exp)} bytes")

# Sum both - generator will be computed on-demand
print(f"Sum of list: {sum(list_comp)}")
print(f"Sum of generator: {sum(gen_exp)}")

# Note: The generator is now exhausted
print(f"Sum of generator again: {sum(gen_exp)}")  # 0 (generator is exhausted)
```

### Chaining Generators

```python
def process_items(items):
    # Chain of generator expressions
    items = (item for item in items)  # Identity transformation
    items = (item * 2 for item in items)  # Double each item
    items = (item for item in items if item % 3 == 0)  # Keep multiples of 3
    return items

numbers = [1, 2, 3, 4, 5]
processed = process_items(numbers)
print(list(processed))  # [6, 12]
```

## The functools Module

The `functools` module provides higher-order functions and operations on callable objects.

### functools.reduce

```python
from functools import reduce

# Calculate product
numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120

# Find maximum
maximum = reduce(lambda x, y: x if x > y else y, numbers)
print(maximum)  # 5

# Custom reduce implementation
def my_reduce(function, iterable, initializer=None):
    it = iter(iterable)
    if initializer is None:
        try:
            value = next(it)
        except StopIteration:
            raise TypeError("reduce() of empty sequence with no initial value")
    else:
        value = initializer
    
    for element in it:
        value = function(value, element)
    return value

print(my_reduce(lambda x, y: x + y, numbers))  # 15
```

### functools.partial

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Create specialized functions
square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(4))  # 16
print(cube(4))    # 64

# Use with built-in functions
from operator import add, mul
increment = partial(add, 1)
double = partial(mul, 2)

print(increment(10))  # 11
print(double(10))     # 20
```

### functools.wraps

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # Preserves function metadata
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Say hello to someone."""
    return f"Hello, {name}!"

print(greet("World"))  # Hello, World!
print(greet.__name__)  # greet (preserved by @wraps)
print(greet.__doc__)   # Say hello to someone. (preserved by @wraps)
```

### functools.singledispatch

```python
from functools import singledispatch

@singledispatch
def process(arg):
    print(f"Processing a {type(arg).__name__} argument")
    return arg

@process.register
def _(arg: int):
    print(f"Processing an integer: {arg}")
    return arg * 2

@process.register
def _(arg: str):
    print(f"Processing a string: {arg}")
    return arg.upper()

@process.register(list)
def _(arg):
    print(f"Processing a list with {len(arg)} elements")
    return arg[::-1]  # Reverse the list

print(process(10))        # Processing an integer: 10 -> 20
print(process("hello"))   # Processing a string: hello -> HELLO
print(process([1, 2, 3])) # Processing a list with 3 elements -> [3, 2, 1]
print(process(1.5))       # Processing a float argument -> 1.5
```

## The itertools Module

The `itertools` module provides functions for creating efficient iterators.

### itertools.count, cycle, repeat

```python
import itertools

# Count - infinite sequence starting at a value
counter = itertools.count(start=10, step=2)
print([next(counter) for _ in range(5)])  # [10, 12, 14, 16, 18]

# Cycle - infinitely repeat a sequence
cycler = itertools.cycle(['A', 'B', 'C'])
print([next(cycler) for _ in range(7)])  # ['A', 'B', 'C', 'A', 'B', 'C', 'A']

# Repeat - repeat a value n times (or infinitely)
repeater = itertools.repeat('X', 3)
print(list(repeater))  # ['X', 'X', 'X']
```

### itertools.chain, zip_longest, product

```python
# Chain - combine multiple iterables
combined = itertools.chain([1, 2, 3], ['A', 'B', 'C'], [True, False])
print(list(combined))  # [1, 2, 3, 'A', 'B', 'C', True, False]

# zip_longest - like zip but with fillvalue for shorter iterables
list1 = [1, 2, 3]
list2 = ['a', 'b']
print(list(zip(list1, list2)))  # [(1, 'a'), (2, 'b')] - stops at shortest
print(list(itertools.zip_longest(list1, list2, fillvalue='*')))  # [(1, 'a'), (2, 'b'), (3, '*')]

# Product - Cartesian product
print(list(itertools.product('AB', '12')))  # [('A', '1'), ('A', '2'), ('B', '1'), ('B', '2')]
print(list(itertools.product([0, 1], repeat=3)))  # All 3-digit binary numbers
```

### itertools.permutations, combinations, combinations_with_replacement

```python
# Permutations - all possible orderings
print(list(itertools.permutations('ABC', 2)))  # [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]

# Combinations - all possible selections without replacement
print(list(itertools.combinations('ABC', 2)))  # [('A', 'B'), ('A', 'C'), ('B', 'C')]

# Combinations with replacement - all possible selections with replacement
print(list(itertools.combinations_with_replacement('ABC', 2)))
# [('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]
```

### itertools.groupby, accumulate, filterfalse

```python
# groupby - group consecutive items by a key function
words = ['apple', 'ant', 'banana', 'bat', 'cat']
grouped = itertools.groupby(sorted(words), key=lambda x: x[0])
for key, group in grouped:
    print(key, list(group))
# a ['ant', 'apple']
# b ['banana', 'bat']
# c ['cat']

# accumulate - running totals
print(list(itertools.accumulate([1, 2, 3, 4, 5])))  # [1, 3, 6, 10, 15]
# With custom function
print(list(itertools.accumulate([1, 2, 3, 4, 5], lambda x, y: x * y)))  # [1, 2, 6, 24, 120]

# filterfalse - opposite of filter
print(list(itertools.filterfalse(lambda x: x % 2 == 0, range(10))))  # [1, 3, 5, 7, 9]
```

## Real-World Applications

### Data Processing Pipeline

```python
# Process a list of customer records
customers = [
    {'id': 1, 'name': 'Alice', 'age': 24, 'active': True},
    {'id': 2, 'name': 'Bob', 'age': 19, 'active': False},
    {'id': 3, 'name': 'Charlie', 'age': 32, 'active': True},
    {'id': 4, 'name': 'Diana', 'age': 45, 'active': True},
    {'id': 5, 'name': 'Eve', 'age': 17, 'active': False}
]

# Functional approach to data processing
def is_adult(customer):
    return customer['age'] >= 18

def is_active(customer):
    return customer['active']

def get_name(customer):
    return customer['name']

def format_customer(name):
    return f"Customer: {name}"

# Chain of operations
def process_customers(customers):
    return list(map(
        format_customer,
        map(
            get_name,
            filter(
                is_active,
                filter(
                    is_adult,
                    customers
                )
            )
        )
    ))

result = process_customers(customers)
print(result)  # ['Customer: Alice', 'Customer: Charlie', 'Customer: Diana']

# Alternative with pipe-like composition
def pipe(data, *funcs):
    result = data
    for func in funcs:
        result = func(result)
    return result

def filter_adults(customers):
    return filter(is_adult, customers)

def filter_active(customers):
    return filter(is_active, customers)

def extract_names(customers):
    return map(get_name, customers)

def format_names(names):
    return map(format_customer, names)

# More readable chaining
result2 = pipe(
    customers,
    filter_adults,
    filter_active,
    extract_names,
    format_names,
    list
)
print(result2)  # ['Customer: Alice', 'Customer: Charlie', 'Customer: Diana']
```

### Event Handling System

```python
# Create a simple event system with functional programming
def create_event_system():
    handlers = {}
    
    def register(event_type, handler):
        if event_type not in handlers:
            handlers[event_type] = []
        handlers[event_type].append(handler)
        return lambda: handlers[event_type].remove(handler)  # Return unregister function
    
    def emit(event_type, *args, **kwargs):
        if event_type in handlers:
            for handler in handlers[event_type]:
                handler(*args, **kwargs)
    
    def get_handlers():
        return {k: len(v) for k, v in handlers.items()}
    
    return register, emit, get_handlers

register, emit, get_handlers = create_event_system()

# Define event handlers
def on_user_login(user_id, username):
    print(f"User logged in: {username} (ID: {user_id})")

def on_user_logout(user_id, username):
    print(f"User logged out: {username} (ID: {user_id})")

def notify_admin_login(user_id, username):
    if username == "admin":
        print(f"ALERT: Admin login detected: {username} (ID: {user_id})")

# Register handlers
unregister_logout = register("user:logout", on_user_logout)
register("user:login", on_user_login)
register("user:login", notify_admin_login)

# Emit events
emit("user:login", 1, "alice")      # User logged in: alice (ID: 1)
emit("user:login", 2, "admin")      # User logged in: admin (ID: 2)
                                    # ALERT: Admin login detected: admin (ID: 2)
emit("user:logout", 1, "alice")     # User logged out: alice (ID: 1)

# Unregister a handler
unregister_logout()
emit("user:logout", 2, "admin")     # Nothing happens - handler was removed

# Check registered handlers
print(get_handlers())  # {'user:logout': 0, 'user:login': 2}
```

### Configuration Management

```python
# Functional approach to config management
def make_config(defaults=None):
    """Create a configuration manager with functional operations."""
    config = defaults.copy() if defaults else {}
    
    def get(key, default=None):
        return config.get(key, default)
    
    def set(key, value):
        config[key] = value
        return get  # Enable chaining
    
    def update(new_values):
        config.update(new_values)
        return get  # Enable chaining
    
    def delete(key):
        if key in config:
            del config[key]
        return get  # Enable chaining
    
    def to_dict():
        return config.copy()
    
    # Return functions dictionary
    return {
        'get': get,
        'set': set,
        'update': update,
        'delete': delete,
        'to_dict': to_dict
    }

# Create a config manager with defaults
config = make_config({
    'debug': False,
    'port': 8080,
    'host': 'localhost',
    'max_connections': 100
})

# Use the functional interface
print(config['get']('port'))  # 8080

# Chained operations
new_port = config['set']('port', 9000)('port')
print(new_port)  # 9000

# Update multiple values and get one
host = config['update']({
    'host': '0.0.0.0',
    'workers': 4
})('host')
print(host)  # 0.0.0.0

# Get full config
print(config['to_dict']())
# {'debug': False, 'port': 9000, 'host': '0.0.0.0', 'max_connections': 100, 'workers': 4}
```

## Best Practices

### 1. Prefer Pure Functions

```python
# Bad: Impure function with side effects
total = 0
def add_to_total(value):
    global total
    total += value
    return total

# Good: Pure function without side effects
def add(a, b):
    return a + b

# Usage
running_total = 0
running_total = add(running_total, 5)  # 5
running_total = add(running_total, 10)  # 15
```

### 2. Use Function Composition

```python
# Bad: Nested function calls are hard to read
result = h(g(f(x)))

# Good: Use function composition
from functools import reduce
def compose(*functions):
    return reduce(lambda f, g: lambda x: f(g(x)), functions, lambda x: x)

composed = compose(h, g, f)
result = composed(x)
```

### 3. Avoid Mutating Data

```python
# Bad: Mutating data
def add_user(users_list, user):
    users_list.append(user)  # Modifies the input list
    return users_list

# Good: Creating new data
def add_user_pure(users_list, user):
    return users_list + [user]  # Returns a new list
```

### 4. Use Higher-Order Functions

```python
numbers = [1, 2, 3, 4, 5]

# Bad: Imperative style
squared = []
for num in numbers:
    squared.append(num ** 2)

# Good: Functional style with higher-order function
squared = list(map(lambda x: x ** 2, numbers))

# Even better: List comprehension (more Pythonic)
squared = [x ** 2 for x in numbers]
```

### 5. Prefer List Comprehensions for Simple Transformations

```python
# Good: List comprehension for simple transformations
names = ["alice", "bob", "charlie"]
capitalized = [name.capitalize() for name in names]

# Good: Use map for more complex functions
def complex_transform(name):
    # Imagine this is a more complex operation
    return f"User: {name.capitalize()}"

transformed = list(map(complex_transform, names))
```

### 6. Use Generator Expressions for Large Datasets

```python
# Bad: Loading all data into memory
def process_large_file(filename):
    with open(filename) as f:
        lines = f.readlines()  # Loads entire file into memory
    
    processed = [line.strip().upper() for line in lines]
    return processed

# Good: Process data lazily with generators
def process_large_file_lazy(filename):
    with open(filename) as f:
        for line in f:  # Iterates one line at a time
            yield line.strip().upper()
```

### 7. Cache Expensive Function Calls

```python
# Bad: Recalculating expensive results
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Good: Using memoization
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci_cached(n):
    if n <= 1:
        return n
    return fibonacci_cached(n-1) + fibonacci_cached(n-2)
```

### 8. Balance Functional and Imperative Styles

```python
# Overly functional (may be hard to read)
result = reduce(lambda x, y: x + y, 
                map(lambda x: x ** 2, 
                    filter(lambda x: x % 2 == 0, range(10))))

# More balanced approach
evens = [x for x in range(10) if x % 2 == 0]
squares = [x ** 2 for x in evens]
result = sum(squares)
```

### 9. Document Function Contracts

```python
def find_user(users, user_id):
    """
    Find a user by their ID.
    
    Args:
        users: List of user dictionaries with 'id' keys
        user_id: Integer ID to search for
    
    Returns:
        The user dictionary if found, None otherwise
    
    This function does not modify the input list.
    """
    for user in users:
        if user['id'] == user_id:
            return user
    return None
```

### 10. Use Named Functions for Clarity

```python
# Less clear with lambdas
data = [1, 2, 3, 4, 5]
result = list(filter(lambda x: x % 2 == 0, map(lambda x: x * 2, data)))

# More clear with named functions
def double(x):
    return x * 2

def is_even(x):
    return x % 2 == 0

result = list(filter(is_even, map(double, data)))
```