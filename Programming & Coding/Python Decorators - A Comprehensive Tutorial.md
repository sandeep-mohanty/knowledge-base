# Python Decorators: A Comprehensive Tutorial

## Table of Contents
1. [Introduction to Decorators](#introduction-to-decorators)
2. [Basic Decorator Syntax](#basic-decorator-syntax)
3. [Decorators with Arguments](#decorators-with-arguments)
4. [Function Decorators with Parameters](#function-decorators-with-parameters)
5. [Class Decorators](#class-decorators)
6. [Method Decorators](#method-decorators)
7. [Multiple Decorators](#multiple-decorators)
8. [Built-in Decorators](#built-in-decorators)
9. [Functional Tools for Decorators](#functional-tools-for-decorators)
10. [Decorator Patterns and Use Cases](#decorator-patterns-and-use-cases)
11. [Best Practices](#best-practices)
12. [Debugging Decorators](#debugging-decorators)

## Introduction to Decorators

Decorators are a powerful feature in Python that allow you to modify the behavior of functions or methods without changing their source code. They use the concept of higher-order functions (functions that take other functions as arguments and/or return functions).

### Key Concepts:

- **Higher-Order Functions**: Functions that operate on other functions
- **Function Wrapping**: Modifying or enhancing a function without changing its definition
- **Syntactic Sugar**: The `@decorator` syntax that makes applying decorators cleaner
- **Metaprogramming**: Code that manipulates code (decorators are a form of metaprogramming)

### Why Use Decorators?

- **Code Reuse**: Apply the same enhancements to multiple functions
- **Separation of Concerns**: Keep core function logic separate from peripheral concerns
- **Clean Code**: Avoid repetitive boilerplate within functions
- **Cross-Cutting Concerns**: Handle aspects like logging, timing, authentication across multiple functions

## Basic Decorator Syntax

### Functions as First-Class Objects

In Python, functions are first-class objects, which means they can be:
- Assigned to variables
- Passed as arguments to other functions
- Returned from other functions
- Stored in data structures

```python
def greet(name):
    return f"Hello, {name}!"

# Assign function to a variable
greeting_function = greet

# Call the function through the variable
print(greeting_function("Alice"))  # Hello, Alice!
```

### A Simple Decorator

```python
def simple_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

def say_hello():
    print("Hello!")

# Apply the decorator manually
decorated_hello = simple_decorator(say_hello)
decorated_hello()
# Output:
# Something is happening before the function is called.
# Hello!
# Something is happening after the function is called.
```

### Using the @ Syntax

```python
def simple_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

# Apply decorator with @ syntax
@simple_decorator
def say_hello():
    print("Hello!")

say_hello()
# Output:
# Something is happening before the function is called.
# Hello!
# Something is happening after the function is called.
```

### Decorating Functions with Arguments

```python
def decorator_with_args(func):
    def wrapper(*args, **kwargs):
        print(f"Arguments: {args}, Keyword arguments: {kwargs}")
        return func(*args, **kwargs)
    return wrapper

@decorator_with_args
def add(a, b):
    return a + b

result = add(3, 5)
print(f"Result: {result}")
# Output:
# Arguments: (3, 5), Keyword arguments: {}
# Result: 8

@decorator_with_args
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("World"))
# Output:
# Arguments: ('World',), Keyword arguments: {'greeting': 'Hello'}
# Hello, World!
```

### Preserving Function Metadata

```python
import functools

def better_decorator(func):
    @functools.wraps(func)  # Preserves the original function's metadata
    def wrapper(*args, **kwargs):
        """Wrapper function"""
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@better_decorator
def add(a, b):
    """Add two numbers and return the result."""
    return a + b

# Without @functools.wraps, this would show wrapper's name and docstring
print(add.__name__)  # add
print(add.__doc__)   # Add two numbers and return the result.
```

## Decorators with Arguments

Decorators can also take arguments, which requires an additional level of nesting.

### Decorator Factory

```python
def repeat(n=1):
    """Repeat the decorated function n times"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(n):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(3)
def say_hello(name):
    return f"Hello, {name}!"

print(say_hello("Alice"))
# Output: ['Hello, Alice!', 'Hello, Alice!', 'Hello, Alice!']

# Can also be used without arguments (uses default n=1)
@repeat()
def say_hi(name):
    return f"Hi, {name}!"

print(say_hi("Bob"))
# Output: ['Hi, Bob!']
```

### Real-World Example: Rate Limiting

```python
import time
import functools
from collections import deque

def rate_limit(max_calls, period):
    """Limit function to max_calls per period seconds"""
    calls = deque(maxlen=max_calls)
    
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            
            # Remove calls older than 'period'
            while calls and calls[0] < now - period:
                calls.popleft()
                
            # Check if we've reached the maximum calls
            if len(calls) >= max_calls:
                raise Exception(f"Rate limit exceeded. Max {max_calls} calls per {period} seconds.")
                
            result = func(*args, **kwargs)
            calls.append(now)
            return result
        return wrapper
    return decorator

@rate_limit(max_calls=3, period=60)
def api_call(endpoint):
    print(f"Calling API endpoint: {endpoint}")
    return {"status": "success"}

# Try to call the function multiple times
for i in range(5):
    try:
        api_call(f"/api/resource/{i}")
    except Exception as e:
        print(f"Exception: {e}")
```

## Function Decorators with Parameters

### Combining Function Parameters and Decorator Parameters

```python
def validate_types(param_types):
    """Validate the types of function arguments"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Check args types
            for arg, expected_type in zip(args, param_types):
                if not isinstance(arg, expected_type):
                    raise TypeError(f"Expected {expected_type}, got {type(arg)}")
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types([str, int])
def process_data(name, age):
    return f"{name} is {age} years old"

# Valid call
print(process_data("Alice", 30))  # Alice is 30 years old

# Invalid calls
try:
    print(process_data(123, 30))  # TypeError
except TypeError as e:
    print(f"Error: {e}")

try:
    print(process_data("Bob", "thirty"))  # TypeError
except TypeError as e:
    print(f"Error: {e}")
```

### Passing Arguments to Both Levels

```python
def format_output(prefix='Result', suffix=''):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            return f"{prefix}: {result} {suffix}".strip()
        return wrapper
    return decorator

@format_output(prefix='Sum', suffix='units')
def add(a, b):
    return a + b

print(add(5, 3))  # Sum: 8 units

@format_output(prefix='Hello')
def get_name():
    return "World"

print(get_name())  # Hello: World
```

## Class Decorators

Class decorators are applied to classes instead of functions.

### Basic Class Decorator

```python
def add_greeting(cls):
    cls.greet = lambda self: f"Hello, I'm {self.name}!"
    return cls

@add_greeting
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Alice")
print(p.greet())  # Hello, I'm Alice!
```

### Modifying Class Behavior

```python
def singleton(cls):
    """Make a class a Singleton class (only one instance)"""
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self, host, port):
        self.host = host
        self.port = port
        print(f"Connecting to database at {host}:{port}")
        
    def query(self, sql):
        print(f"Executing SQL: {sql}")
        return ["result1", "result2"]

# First time creates the connection
db1 = DatabaseConnection("localhost", 5432)
# Second time reuses the same instance
db2 = DatabaseConnection("localhost", 5432)

print(db1 is db2)  # True
```

### Class Factory Decorator

```python
def add_methods(*methods):
    def decorator(cls):
        for method in methods:
            setattr(cls, method.__name__, method)
        return cls
    return decorator

# Define standalone methods to add
def bark(self):
    return f"{self.name} says Woof!"

def fetch(self, item):
    return f"{self.name} fetched the {item}!"

@add_methods(bark, fetch)
class Dog:
    def __init__(self, name):
        self.name = name

dog = Dog("Buddy")
print(dog.bark())         # Buddy says Woof!
print(dog.fetch("ball"))  # Buddy fetched the ball!
```

## Method Decorators

Method decorators work on methods inside classes.

### Decorating Instance Methods

```python
def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args[1:]} and {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

class Calculator:
    def __init__(self, name):
        self.name = name
    
    @log_calls
    def add(self, a, b):
        """Add two numbers"""
        return a + b
    
    @log_calls
    def multiply(self, a, b):
        """Multiply two numbers"""
        return a * b

calc = Calculator("MyCalc")
calc.add(3, 5)
# Output:
# Calling add with (3, 5) and {}
# add returned 8

calc.multiply(2, 6)
# Output:
# Calling multiply with (2, 6) and {}
# multiply returned 12
```

### Decorating Special Methods

```python
def validate_positive(func):
    @functools.wraps(func)
    def wrapper(self, value):
        if value <= 0:
            raise ValueError("Value must be positive")
        return func(self, value)
    return wrapper

class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
    
    @validate_positive
    def deposit(self, amount):
        self.balance += amount
        return self.balance
    
    @validate_positive
    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        return self.balance

account = BankAccount("Alice", 100)
print(account.deposit(50))  # 150

try:
    account.deposit(-10)  # ValueError: Value must be positive
except ValueError as e:
    print(f"Error: {e}")
```

## Multiple Decorators

You can apply multiple decorators to a single function. They are applied from bottom to top (innermost to outermost).

```python
def bold(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

def underline(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return f"<u>{func(*args, **kwargs)}</u>"
    return wrapper

@bold
@italic
@underline
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
# Output: <b><i><u>Hello, Alice!</u></i></b>
```

### Execution Order

```python
def decorator1(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator 1 - Before")
        result = func(*args, **kwargs)
        print("Decorator 1 - After")
        return result
    return wrapper

def decorator2(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("Decorator 2 - Before")
        result = func(*args, **kwargs)
        print("Decorator 2 - After")
        return result
    return wrapper

@decorator1
@decorator2
def say_hello():
    print("Hello!")

say_hello()
# Output:
# Decorator 1 - Before
# Decorator 2 - Before
# Hello!
# Decorator 2 - After
# Decorator 1 - After
```

## Built-in Decorators

Python includes several built-in decorators that are commonly used.

### @property

Transforms a method into a read-only attribute.

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        """Get the circle's radius"""
        return self._radius
    
    @property
    def diameter(self):
        """Calculate the circle's diameter"""
        return 2 * self._radius
    
    @property
    def area(self):
        """Calculate the circle's area"""
        return 3.14159 * self._radius ** 2

circle = Circle(5)
print(circle.radius)   # 5
print(circle.diameter) # 10
print(circle.area)     # 78.53975

# This will raise an AttributeError
try:
    circle.radius = 10
except AttributeError as e:
    print(f"Error: {e}")
```

### @classmethod and @staticmethod

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    def display(self):
        return f"{self.year}-{self.month:02d}-{self.day:02d}"
    
    @classmethod
    def from_string(cls, date_string):
        """Create a Date object from a string (YYYY-MM-DD)"""
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)
    
    @staticmethod
    def is_valid_date(year, month, day):
        """Check if a date is valid"""
        if year < 1:
            return False
        if month < 1 or month > 12:
            return False
        if day < 1 or day > 31:
            return False
        return True

# Using instance method
date = Date(2023, 9, 15)
print(date.display())  # 2023-09-15

# Using class method
date_from_str = Date.from_string("2023-10-20")
print(date_from_str.display())  # 2023-10-20

# Using static method
print(Date.is_valid_date(2023, 13, 5))  # False
print(Date.is_valid_date(2023, 6, 15))   # True
```

### @staticmethod vs @classmethod

```python
class MathOperations:
    multiplier = 2
    
    def __init__(self, value):
        self.value = value
    
    def multiply(self):
        return self.value * self.multiplier
    
    @classmethod
    def set_multiplier(cls, new_multiplier):
        cls.multiplier = new_multiplier
        return cls
    
    @staticmethod
    def add(a, b):
        return a + b

# Using class method to modify class variable
print(MathOperations.multiplier)  # 2
MathOperations.set_multiplier(5)
print(MathOperations.multiplier)  # 5

# Create instance after changing multiplier
math_op = MathOperations(10)
print(math_op.multiply())  # 50

# Static method doesn't need a class instance
print(MathOperations.add(3, 7))  # 10
```

## Functional Tools for Decorators

The `functools` module provides useful tools for working with decorators.

### @functools.wraps

Preserves metadata of the decorated function.

```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def my_function():
    """My function docstring"""
    pass

print(my_function.__name__)  # my_function
print(my_function.__doc__)   # My function docstring
```

### @functools.lru_cache

Caches function results to avoid redundant calculations.

```python
import functools
import time

@functools.lru_cache(maxsize=128)
def fibonacci(n):
    if n <= 1:
        return n
    print(f"Computing fibonacci({n})")
    return fibonacci(n-1) + fibonacci(n-2)

start = time.time()
print(fibonacci(30))  # Only unique calls are computed
end = time.time()
print(f"Time taken: {end - start:.6f} seconds")

# Check cache info
print(fibonacci.cache_info())
```

### @functools.singledispatch

Enables function overloading based on the type of the first argument.

```python
import functools

@functools.singledispatch
def process(arg):
    print(f"Default implementation for {type(arg)}")
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

print(process(10))        # Processing an integer: 10
print(process("hello"))   # Processing a string: hello
print(process([1, 2, 3])) # Processing a list with 3 elements
print(process(1.5))       # Default implementation for <class 'float'>
```

## Decorator Patterns and Use Cases

### Timing Functions

```python
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} took {end_time - start_time:.6f} seconds to run")
        return result
    return wrapper

@timer
def slow_function(n):
    """A deliberately slow function"""
    time.sleep(n)
    return f"Slept for {n} seconds"

print(slow_function(1.5))
# slow_function took 1.500123 seconds to run
# Slept for 1.5 seconds
```

### Retry Mechanism

```python
import functools
import time
import random

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            attempts = 0
            while attempts < max_attempts:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    attempts += 1
                    if attempts == max_attempts:
                        raise
                    print(f"Attempt {attempts} failed with error: {e}. Retrying in {delay} seconds...")
                    time.sleep(delay)
            return None  # Should never reach here
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5)
def unstable_function():
    """A function that sometimes fails"""
    if random.random() < 0.7:  # 70% chance of failure
        raise ValueError("Random failure")
    return "Success!"

try:
    result = unstable_function()
    print(result)
except ValueError as e:
    print(f"Function failed after all retries: {e}")
```

### Authentication and Authorization

```python
import functools

def login_required(func):
    @functools.wraps(func)
    def wrapper(user, *args, **kwargs):
        if not user.is_authenticated:
            raise PermissionError("You must be logged in to access this resource")
        return func(user, *args, **kwargs)
    return wrapper

def admin_required(func):
    @functools.wraps(func)
    def wrapper(user, *args, **kwargs):
        if not user.is_admin:
            raise PermissionError("Administrator privileges required")
        return func(user, *args, **kwargs)
    return wrapper

class User:
    def __init__(self, username, is_authenticated=False, is_admin=False):
        self.username = username
        self.is_authenticated = is_authenticated
        self.is_admin = is_admin

@login_required
def view_profile(user):
    return f"Profile for {user.username}"

@login_required
@admin_required
def admin_dashboard(user):
    return f"Admin dashboard for {user.username}"

# Regular user
regular_user = User("alice", is_authenticated=True)
print(view_profile(regular_user))  # Profile for alice

try:
    print(admin_dashboard(regular_user))  # PermissionError
except PermissionError as e:
    print(f"Error: {e}")

# Admin user
admin_user = User("admin", is_authenticated=True, is_admin=True)
print(admin_dashboard(admin_user))  # Admin dashboard for admin
```

### Memoization (Caching)

```python
import functools

def memoize(func):
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Create a key that can be used as a dictionary key
        key = str(args) + str(kwargs)
        
        if key not in cache:
            cache[key] = func(*args, **kwargs)
            print(f"Calculated result for {key}")
        else:
            print(f"Using cached result for {key}")
            
        return cache[key]
    
    return wrapper

@memoize
def expensive_calculation(a, b):
    print(f"Performing expensive calculation for {a}, {b}")
    # Simulate expensive operation
    import time
    time.sleep(1)
    return a * b

print(expensive_calculation(5, 3))  # Calculates
print(expensive_calculation(5, 3))  # Uses cache
print(expensive_calculation(2, 8))  # Calculates
print(expensive_calculation(5, 3))  # Uses cache
```

## Best Practices

### 1. Always Use @functools.wraps

```python
import functools

# Good practice
def good_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# Bad practice
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@good_decorator
def function1():
    """This is function1"""
    pass

@bad_decorator
def function2():
    """This is function2"""
    pass

print(function1.__name__, function1.__doc__)  # function1 This is function1
print(function2.__name__, function2.__doc__)  # wrapper None
```

### 2. Keep Decorators Simple and Focused

```python
# Good: Single responsibility
def log_calls(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

def validate_args(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if len(args) == 0:
            raise ValueError("At least one argument is required")
        return func(*args, **kwargs)
    return wrapper

# Use multiple decorators instead of one complex decorator
@log_calls
@validate_args
def process_data(data):
    return data
```

### 3. Handle Decorator Arguments Consistently

```python
def decorator_factory(arg1=None, arg2=None):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Use arg1, arg2
            return func(*args, **kwargs)
        return wrapper
    
    # Handle the case when decorator is used without arguments
    if callable(arg1):
        func = arg1
        arg1 = None
        return decorator(func)
    
    return decorator

# Can be used both ways
@decorator_factory
def func1():
    pass

@decorator_factory(arg1=10, arg2=20)
def func2():
    pass
```

### 4. Avoid Stateful Decorators (Usually)

```python
# Problematic: Shared state between all decorated functions
def stateful_counter(func):
    counter = [0]  # Using a list as a mutable container
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        counter[0] += 1
        print(f"Call count: {counter[0]}")
        return func(*args, **kwargs)
    return wrapper

# Better: Use a class to encapsulate state
class CountCalls:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.call_count = 0
    
    def __call__(self, *args, **kwargs):
        self.call_count += 1
        print(f"{self.func.__name__} has been called {self.call_count} times")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    return "Hello"

say_hello()  # say_hello has been called 1 times
say_hello()  # say_hello has been called 2 times
```

## Debugging Decorators

Debugging decorated functions can be challenging. Here are some strategies:

### 1. Bypass the Decorator Temporarily

```python
def debug_decorator(func):
    # Set this to True to bypass the decorator logic
    DEBUG_MODE = False
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if DEBUG_MODE:
            return func(*args, **kwargs)
        else:
            # Decorator logic
            print("Before function call")
            result = func(*args, **kwargs)
            print("After function call")
            return result
    return wrapper

@debug_decorator
def problematic_function(x):
    return x / 0  # Will raise an error

try:
    problematic_function(10)
except ZeroDivisionError:
    print("Caught division by zero error")
```

### 2. Add Debug Logging

```python
import functools
import logging

logging.basicConfig(level=logging.DEBUG)

def log_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logging.debug(f"Calling {func.__name__} with args: {args}, kwargs: {kwargs}")
        try:
            result = func(*args, **kwargs)
            logging.debug(f"{func.__name__} returned: {result}")
            return result
        except Exception as e:
            logging.error(f"{func.__name__} raised: {e}")
            raise
    return wrapper

@log_decorator
def divide(a, b):
    return a / b

try:
    divide(10, 2)
    divide(10, 0)
except ZeroDivisionError:
    print("Caught division by zero error")
```

### 3. Inspecting Decorated Functions

```python
import inspect
import functools

def verbose_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    
    # Add attribute to indicate this function is decorated
    wrapper.is_decorated = True
    return wrapper

@verbose_decorator
def example_function(a, b):
    """Example function docstring"""
    return a + b

# Inspect the function
print(f"Name: {example_function.__name__}")
print(f"Doc: {example_function.__doc__}")
print(f"Is decorated: {getattr(example_function, 'is_decorated', False)}")
print(f"Signature: {inspect.signature(example_function)}")
print(f"Source: {inspect.getsource(example_function)}")
```

### 4. Using Python's Trace Features

```python
import sys
import functools

def traced_function(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Enable tracing
        sys.settrace(trace_calls)
        result = func(*args, **kwargs)
        # Disable tracing
        sys.settrace(None)
        return result
    return wrapper

def trace_calls(frame, event, arg):
    if event == 'call':
        print(f"Calling {frame.f_code.co_name} at line {frame.f_lineno}")
    return trace_calls

@traced_function
def recursive_function(n):
    if n <= 1:
        return 1
    return n * recursive_function(n-1)

print(recursive_function(5))
```