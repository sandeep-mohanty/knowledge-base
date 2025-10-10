# Python Modules and Packages Tutorial

## Table of Contents
1. [Introduction](#introduction)
2. [What are Modules?](#what-are-modules)
3. [Creating and Using Modules](#creating-and-using-modules)
4. [The import Statement](#the-import-statement)
5. [What are Packages?](#what-are-packages)
6. [Creating Packages](#creating-packages)
7. [The __init__.py File](#the-__init__py-file)
8. [Importing from Packages](#importing-from-packages)
9. [The __name__ Variable](#the-__name__-variable)
10. [Best Practices](#best-practices)

## Introduction

Python modules and packages are fundamental concepts for organizing and structuring code. They help you write maintainable, reusable, and scalable applications by breaking down large programs into smaller, manageable pieces.

## What are Modules?

A module is simply a Python file containing Python code. It can define functions, classes, and variables, and can also include runnable code. Modules help you organize related code into separate files.

### Benefits of Using Modules:
- **Code reusability**: Write once, use multiple times
- **Namespace separation**: Avoid naming conflicts
- **Maintainability**: Easier to manage and update code
- **Collaboration**: Multiple developers can work on different modules

## Creating and Using Modules

### Creating a Module

Create a file named `math_operations.py`:

```python
# math_operations.py

def add(a, b):
    """Add two numbers"""
    return a + b

def subtract(a, b):
    """Subtract b from a"""
    return a - b

def multiply(a, b):
    """Multiply two numbers"""
    return a * b

def divide(a, b):
    """Divide a by b"""
    if b != 0:
        return a / b
    else:
        raise ValueError("Cannot divide by zero")

PI = 3.14159
```

### Using the Module

Create another file in the same directory:

```python
# main.py

import math_operations

result = math_operations.add(5, 3)
print(f"5 + 3 = {result}")

print(f"PI value: {math_operations.PI}")
```

## The import Statement

Python provides several ways to import modules:

### 1. Basic Import

```python
import module_name
```

### 2. Import with Alias

```python
import module_name as alias
```

Example:
```python
import math_operations as math_ops

result = math_ops.multiply(4, 5)
```

### 3. Import Specific Items

```python
from module_name import item1, item2
```

Example:
```python
from math_operations import add, PI

result = add(10, 20)  # No need for module prefix
print(PI)
```

### 4. Import All Items

```python
from module_name import *
```

**Note**: This is generally discouraged as it can lead to namespace pollution.

### 5. Import from Submodules

```python
from package.subpackage import module
```

## What are Packages?

A package is a way of organizing related modules into a directory hierarchy. It's essentially a directory containing Python files and a special `__init__.py` file.

### Package Structure Example:

```
my_package/
│   __init__.py
│   module1.py
│   module2.py
│
└───subpackage/
    │   __init__.py
    │   module3.py
    │   module4.py
```

## Creating Packages

Let's create a simple package for a calculator application:

### Step 1: Create Directory Structure

```
calculator/
│   __init__.py
│   basic.py
│   advanced.py
│
└───conversions/
    │   __init__.py
    │   temperature.py
    │   length.py
```

### Step 2: Create Module Files

**calculator/basic.py:**
```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b != 0:
        return a / b
    raise ValueError("Cannot divide by zero")
```

**calculator/advanced.py:**
```python
import math

def power(base, exponent):
    return base ** exponent

def square_root(number):
    return math.sqrt(number)

def factorial(n):
    return math.factorial(n)
```

**calculator/conversions/temperature.py:**
```python
def celsius_to_fahrenheit(celsius):
    return (celsius * 9/5) + 32

def fahrenheit_to_celsius(fahrenheit):
    return (fahrenheit - 32) * 5/9

def celsius_to_kelvin(celsius):
    return celsius + 273.15
```

**calculator/conversions/length.py:**
```python
def meters_to_feet(meters):
    return meters * 3.28084

def feet_to_meters(feet):
    return feet / 3.28084

def kilometers_to_miles(km):
    return km * 0.621371
```

## The __init__.py File

The `__init__.py` file serves several purposes:

1. **Marks a directory as a Python package**
2. **Can be empty or contain initialization code**
3. **Controls what gets imported with `from package import *`**

### Example __init__.py Files:

**calculator/__init__.py:**
```python
"""Calculator package for basic and advanced mathematical operations"""

# Version of the calculator package
__version__ = "1.0.0"

# Import commonly used functions for easier access
from .basic import add, subtract, multiply, divide
from .advanced import power, square_root

# Define what gets imported with "from calculator import *"
__all__ = ['add', 'subtract', 'multiply', 'divide', 'power', 'square_root']
```

**calculator/conversions/__init__.py:**
```python
"""Conversion utilities for temperature and length"""

from .temperature import celsius_to_fahrenheit, fahrenheit_to_celsius
from .length import meters_to_feet, feet_to_meters
```

## Importing from Packages

### Different Ways to Import from Packages:

```python
# Import the entire package
import calculator

# Import specific module from package
from calculator import basic

# Import specific function from module in package
from calculator.basic import add

# Import from subpackage
from calculator.conversions import temperature

# Import specific function from subpackage
from calculator.conversions.temperature import celsius_to_fahrenheit

# Using alias
import calculator.conversions.temperature as temp_conv
```

### Usage Example:

```python
# main.py

# Method 1: Import entire package
import calculator
result = calculator.add(5, 3)  # Works because of __init__.py imports

# Method 2: Import specific module
from calculator import advanced
root = advanced.square_root(16)

# Method 3: Import specific functions
from calculator.conversions.temperature import celsius_to_fahrenheit
temp_f = celsius_to_fahrenheit(25)

# Method 4: Import with alias
import calculator.conversions.length as length_conv
distance = length_conv.meters_to_feet(100)

print(f"5 + 3 = {result}")
print(f"Square root of 16 = {root}")
print(f"25°C = {temp_f}°F")
print(f"100 meters = {distance} feet")
```

## The __name__ Variable

Every Python module has a built-in attribute `__name__`. When a module is run directly, `__name__` is set to `"__main__"`. When imported, it's set to the module's name.

### Example:

```python
# demo_module.py

def greet(name):
    return f"Hello, {name}!"

print(f"Module name: {__name__}")

if __name__ == "__main__":
    # This code only runs when the module is executed directly
    print("This module is being run directly")
    print(greet("World"))
else:
    # This code runs when the module is imported
    print("This module has been imported")
```

When you run `python demo_module.py`:
```
Module name: __main__
This module is being run directly
Hello, World!
```

When you import it in another file:
```python
import demo_module  # Output: Module name: demo_module
                    #         This module has been imported
```

## Best Practices

### 1. Module Naming
- Use lowercase with underscores: `my_module.py`
- Avoid using Python built-in names
- Keep names short but descriptive

### 2. Package Structure
- Organize related functionality together
- Keep packages focused on a single purpose
- Use meaningful directory names

### 3. Imports
- Place all imports at the top of the file
- Order imports: standard library, third-party, local
- Avoid circular imports
- Use absolute imports when possible

### 4. Documentation
- Add docstrings to modules, functions, and classes
- Include a README file for packages
- Document the purpose and usage of each module

### 5. Example of Well-Structured Code:

```python
"""
math_utilities.py

This module provides advanced mathematical utilities.

Author: Your Name
Date: 2024-01-01
"""

import math
from typing import Union

# Constants
EULER_NUMBER = 2.71828


def calculate_compound_interest(
    principal: float,
    rate: float,
    time: int,
    n: int = 1
) -> float:
    """
    Calculate compound interest.
    
    Args:
        principal: Initial amount
        rate: Annual interest rate (as decimal)
        time: Time period in years
        n: Number of times interest is compounded per year
        
    Returns:
        Final amount after compound interest
    """
    amount = principal * (1 + rate/n) ** (n * time)
    return round(amount, 2)


if __name__ == "__main__":
    # Test the function
    result = calculate_compound_interest(1000, 0.05, 2)
    print(f"Compound interest result: ${result}")
```

### 6. Package Distribution Structure

For distributable packages, use this structure:

```
my_project/
│   setup.py
│   README.md
│   LICENSE
│   requirements.txt
│
├───my_package/
│   │   __init__.py
│   │   core.py
│   │   utils.py
│   │
│   └───submodule/
│       │   __init__.py
│       │   helpers.py
│
└───tests/
    │   test_core.py
    │   test_utils.py
```

## Conclusion

Modules and packages are essential tools for organizing Python code. They promote code reusability, maintainability, and help manage complexity in larger projects. By following the practices outlined in this tutorial, you can create well-structured Python applications that are easy to understand, maintain, and extend.

Remember to:
- Keep modules focused on specific functionality
- Use clear and descriptive names
- Document your code properly
- Follow Python's import conventions
- Organize related modules into packages

With these concepts mastered, you're well-equipped to structure your Python projects effectively!