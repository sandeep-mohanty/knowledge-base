# Ways to Optimize Your Code in Python

Python is beloved for its readability and simplicity, but performance isn't always its strongest suit. Thankfully, there are several techniques you can apply to optimize your Python code for better speed and efficiency. This tutorial walks through seven practical ways to optimize your Python code, with examples and tips.

## 1. Use Built-in Functions and Libraries

Python’s built-in functions are implemented in C and are significantly faster than manually written loops. Whenever possible, leverage them.

**✅ Do this:**

```python
result = sum([1, 2, 3, 4])
```

**❌ Not this:**

```python
result = 0
for i in [1, 2, 3, 4]:
    result += i
```

**Tip**: The `math`, `itertools`, and `collections` modules offer optimized utilities.

## 2. Avoid Using Global Variables

Accessing global variables is slower than local ones, and can also introduce unintended side effects.

**✅ Use local variables:**

```python
def compute():
    local_var = 10
    return local_var * 2
```

**❌ Avoid globals:**

```python
global_var = 10
def compute():
    return global_var * 2
```

## 3. Use List Comprehensions Instead of Loops

List comprehensions are not only more concise but also faster than `for` loops for creating lists.

**✅ Do this:**

```python
squares = [x * x for x in range(10)]
```

**❌ Not this:**

```python
squares = []
for x in range(10):
    squares.append(x * x)
```

## 4. Use Generators Instead of Lists for Large Datasets

Generators are memory-efficient and lazy-evaluated, which makes them ideal for handling large datasets.

**✅ Use a generator:**

```python
def count_up_to(n):
    for i in range(n):
        yield i
```

**❌ Avoid building huge lists:**

```python
def count_up_to(n):
    return [i for i in range(n)]
```

## 5. Use Efficient Data Structures

Choosing the right data structure can drastically improve performance.

- Use sets for membership tests instead of lists.
- Use `defaultdict` or `Counter` from `collections` for counting.

**Example:**

```python
from collections import Counter

words = ['apple', 'banana', 'apple']
count = Counter(words)
```

## 6. Profile Your Code

Use profiling tools to identify bottlenecks before optimizing.

**Tools:**

- `cProfile` (built-in)
- `line_profiler`
- `memory_profiler`

**Example:**

```bash
python -m cProfile my_script.py
```

## 7. Use Cython or PyPy for Performance-Critical Code

When native Python isn't fast enough, consider using tools like:

- **Cython** – Compiles Python to C
- **PyPy** – A faster Python interpreter with JIT compilation

**Example with Cython:**

```cython
# my_module.pyx
def compute(int x):
    return x * x
```

Then compile with Cython and use in Python.

## Final Thoughts

Optimization should be driven by real performance needs. Always measure before and after changes. Premature optimization can lead to complex, unreadable code.
