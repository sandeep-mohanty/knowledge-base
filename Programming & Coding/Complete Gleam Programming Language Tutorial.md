# Complete Gleam Programming Language Tutorial

## Table of Contents

1. [Introduction to Gleam](#introduction-to-gleam)
2. [Setting Up Your Environment](#setting-up-your-environment)
3. [Your First Gleam Program](#your-first-gleam-program)
4. [Basic Syntax and Concepts](#basic-syntax-and-concepts)
5. [Data Types](#data-types)
6. [Functions](#functions)
7. [Pattern Matching](#pattern-matching)
8. [Custom Types](#custom-types)
9. [Lists and List Operations](#lists-and-list-operations)
10. [Tuples and Records](#tuples-and-records)
11. [Error Handling with Result](#error-handling-with-result)
12. [Modules and Imports](#modules-and-imports)
13. [Working with Strings](#working-with-strings)
14. [Higher-Order Functions](#higher-order-functions)
15. [Type System Deep Dive](#type-system-deep-dive)
16. [Building a Real Project](#building-a-real-project)
17. [Best Practices](#best-practices)

---

## Introduction to Gleam

**Gleam** is a friendly, functional programming language that compiles to **Erlang** and **JavaScript**. It runs on the battle-tested BEAM virtual machine (the same VM that powers Erlang and Elixir), making it perfect for building scalable, fault-tolerant systems.

### Why Gleam?

- **Type-safe**: Gleam has a powerful static type system that catches errors at compile time
- **Friendly error messages**: Compiler errors are designed to be helpful, not cryptic
- **Immutable by default**: All data is immutable, making concurrent programming easier
- **No null values**: Gleam uses `Option` types instead of null/nil
- **Fast**: Runs on the BEAM VM, known for low-latency and high-throughput
- **Interoperable**: Can use existing Erlang and Elixir libraries

### Key Features

- **Static typing** with type inference
- **Pattern matching** for control flow
- **First-class functions**
- **No exceptions** - uses Result types for error handling
- **Pipe operator** for readable function composition
- **Exhaustive case expressions**

---

## Setting Up Your Environment

### Installation

**On macOS or Linux:**

```bash
# Using Homebrew (macOS)
brew install gleam

# Or download from GitHub releases
wget https://github.com/gleam-lang/gleam/releases/latest/download/gleam-<version>-<platform>.tar.gz
tar -xzf gleam-<version>-<platform>.tar.gz
sudo mv gleam /usr/local/bin/
```

**On Windows:**

```powershell
# Using Scoop
scoop install gleam

# Or download the Windows installer from GitHub releases
```

### Verify Installation

```bash
gleam --version
```

### Create Your First Project

```bash
gleam new my_first_project
cd my_first_project
```

This creates a new Gleam project with the following structure:

```
my_first_project/
├── gleam.toml           # Project configuration
├── src/
│   └── my_first_project.gleam  # Main source file
├── test/
│   └── my_first_project_test.gleam  # Test file
└── README.md
```

---

## Your First Gleam Program

Let's look at the default program created by `gleam new`:

```gleam
import gleam/io

pub fn main() {
  io.println("Hello from my_first_project!")
}
```

### Running Your Program

```bash
gleam run
```

You should see:

```
Hello from my_first_project!
```

### Understanding the Code

- `import gleam/io` - Imports the IO module for input/output operations
- `pub fn main()` - Defines a public function named `main`
- `io.println()` - Calls the `println` function from the `io` module

---

## Basic Syntax and Concepts

### Comments

```gleam
// This is a single-line comment

// Gleam doesn't have multi-line comments
// So you use multiple single-line comments
// like this
```

### Variables and Bindings

In Gleam, you use `let` to bind values to names. All bindings are **immutable**.

```gleam
import gleam/io

pub fn main() {
  // Simple binding
  let name = "Alice"
  let age = 30
  let is_developer = True
  
  io.println(name)
  
  // You can shadow variables (create a new binding with the same name)
  let age = age + 1
  io.debug(age)  // Prints: 31
  
  // This would cause an error (can't reassign):
  // age = 32  // ERROR!
}
```

### Naming Conventions

- **Variables and functions**: `snake_case`
- **Types and constructors**: `PascalCase`
- **Constants**: `snake_case` (same as variables)

```gleam
// Good naming
let user_name = "Bob"
let max_count = 100

pub type UserRole {
  Admin
  Moderator
  RegularUser
}
```

### Assertions

Use `let assert` when you're certain a pattern will match:

```gleam
pub fn main() {
  let numbers = [1, 2, 3]
  
  // Assert that the list has at least one element
  let assert [first, ..rest] = numbers
  
  io.debug(first)  // Prints: 1
}
```

---

## Data Types

### Integers

```gleam
pub fn integer_examples() {
  let positive = 42
  let negative = -17
  let hex = 0xFF        // 255
  let octal = 0o77      // 63
  let binary = 0b1010   // 10
  
  // Arithmetic operations
  let sum = 10 + 5      // 15
  let difference = 10 - 5  // 5
  let product = 10 * 5     // 50
  let quotient = 10 / 3    // 3 (integer division)
  let remainder = 10 % 3   // 1
  
  // Integer comparison
  let is_equal = 5 == 5      // True
  let is_greater = 10 > 5    // True
  let is_less_or_equal = 5 <= 10  // True
}
```

### Floats

```gleam
pub fn float_examples() {
  let pi = 3.14159
  let negative = -2.5
  let scientific = 1.5e10  // 15000000000.0
  
  // Float operations
  let sum = 1.5 +. 2.5      // 4.0 (note the dot!)
  let difference = 5.0 -. 2.5  // 2.5
  let product = 2.0 *. 3.0     // 6.0
  let quotient = 10.0 /. 4.0   // 2.5
  
  // Note: Int and Float operations use different operators
  // Int: +, -, *, /, %
  // Float: +., -., *., /.
}
```

### Booleans

```gleam
pub fn boolean_examples() {
  let is_true = True
  let is_false = False
  
  // Boolean operators
  let and_result = True && False   // False
  let or_result = True || False    // True
  let not_result = !True           // False
  
  // Comparison operators return booleans
  let comparison = 5 > 3           // True
}
```

### Strings

```gleam
import gleam/string
import gleam/io

pub fn string_examples() {
  let greeting = "Hello, World!"
  let multiline = "This is
  a multiline
  string"
  
  // String concatenation
  let full_name = "John" <> " " <> "Doe"  // "John Doe"
  
  // String functions
  let uppercase = string.uppercase("hello")  // "HELLO"
  let lowercase = string.lowercase("HELLO")  // "hello"
  let length = string.length("Gleam")        // 5
  
  // String contains
  let contains = string.contains("Hello World", "World")  // True
  
  io.println(full_name)
}
```

### String Interpolation

Gleam doesn't have built-in string interpolation, but you can concatenate strings:

```gleam
import gleam/int
import gleam/io

pub fn string_interpolation() {
  let name = "Alice"
  let age = 30
  
  // Using concatenation
  let message = "My name is " <> name <> " and I'm " <> int.to_string(age) <> " years old"
  
  io.println(message)
  // Output: My name is Alice and I'm 30 years old
}
```

### Nil (The Unit Type)

Gleam has a `Nil` type for representing the absence of a value:

```gleam
pub fn nil_example() {
  let nothing = Nil
  
  // Often used in functions that perform side effects
  // and don't return meaningful values
}
```

---

## Functions

### Defining Functions

```gleam
import gleam/io

// Public function (can be used from other modules)
pub fn greet(name: String) -> String {
  "Hello, " <> name <> "!"
}

// Private function (only used within this module)
fn calculate_double(x: Int) -> Int {
  x * 2
}

pub fn main() {
  let greeting = greet("Alice")
  io.println(greeting)
  
  let doubled = calculate_double(5)
  io.debug(doubled)  // Prints: 10
}
```

### Function with Multiple Parameters

```gleam
pub fn add(a: Int, b: Int) -> Int {
  a + b
}

pub fn introduce(first_name: String, last_name: String, age: Int) -> String {
  first_name <> " " <> last_name <> " is " <> int.to_string(age) <> " years old"
}
```

### Anonymous Functions (Lambdas)

```gleam
import gleam/list
import gleam/io

pub fn lambda_examples() {
  // Anonymous function syntax
  let double = fn(x) { x * 2 }
  io.debug(double(5))  // Prints: 10
  
  // Using anonymous functions with higher-order functions
  let numbers = [1, 2, 3, 4, 5]
  let doubled = list.map(numbers, fn(x) { x * 2 })
  io.debug(doubled)  // Prints: [2, 4, 6, 8, 10]
  
  // Short-hand syntax with capture
  let tripled = list.map(numbers, fn(x) { x * 3 })
}
```

### Function Captures

Function captures provide a shorthand for creating anonymous functions:

```gleam
import gleam/list
import gleam/io

pub fn add(a: Int, b: Int) -> Int {
  a + b
}

pub fn capture_examples() {
  let numbers = [1, 2, 3, 4, 5]
  
  // Using capture to partially apply a function
  let add_ten = add(10, _)
  io.debug(add_ten(5))  // Prints: 15
  
  // Using captures with list functions
  let plus_ten = list.map(numbers, add(10, _))
  io.debug(plus_ten)  // Prints: [11, 12, 13, 14, 15]
}
```

### Pipe Operator

The pipe operator `|>` makes code more readable by chaining function calls:

```gleam
import gleam/string
import gleam/list
import gleam/io

pub fn pipe_examples() {
  // Without pipe
  let result1 = string.uppercase(string.trim("  hello  "))
  
  // With pipe (more readable!)
  let result2 = 
    "  hello  "
    |> string.trim
    |> string.uppercase
  
  io.debug(result2)  // Prints: "HELLO"
  
  // More complex example
  let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  
  let result = 
    numbers
    |> list.filter(fn(x) { x % 2 == 0 })  // Keep even numbers
    |> list.map(fn(x) { x * 2 })          // Double them
    |> list.take(3)                        // Take first 3
  
  io.debug(result)  // Prints: [4, 8, 12]
}
```

### Labeled Arguments

Functions can have labeled arguments for clarity:

```gleam
import gleam/io

pub fn create_user(name name: String, age age: Int, email email: String) -> String {
  name <> " (" <> int.to_string(age) <> ") - " <> email
}

pub fn main() {
  // Call with labels
  let user = create_user(name: "Alice", age: 30, email: "alice@example.com")
  
  // Order doesn't matter with labels
  let user2 = create_user(email: "bob@example.com", name: "Bob", age: 25)
  
  io.println(user)
}
```

---

## Pattern Matching

Pattern matching is one of Gleam's most powerful features. It allows you to destructure data and handle different cases elegantly.

### Basic Pattern Matching with Case

```gleam
import gleam/io

pub fn describe_number(n: Int) -> String {
  case n {
    0 -> "zero"
    1 -> "one"
    2 -> "two"
    _ -> "many"  // _ is a wildcard that matches anything
  }
}

pub fn main() {
  io.println(describe_number(0))  // "zero"
  io.println(describe_number(5))  // "many"
}
```

### Pattern Matching on Booleans

```gleam
pub fn is_adult(age: Int) -> String {
  case age >= 18 {
    True -> "Adult"
    False -> "Minor"
  }
}
```

### Pattern Matching on Strings

```gleam
pub fn greet_by_language(language: String) -> String {
  case language {
    "english" -> "Hello!"
    "spanish" -> "¡Hola!"
    "french" -> "Bonjour!"
    "german" -> "Guten Tag!"
    _ -> "Hello!"  // Default case
  }
}
```

### Multiple Patterns

```gleam
pub fn classify_number(n: Int) -> String {
  case n {
    0 -> "zero"
    1 | -1 -> "one or negative one"  // Multiple patterns
    n if n > 0 -> "positive"         // Guard clause
    _ -> "negative"
  }
}
```

### Pattern Matching on Tuples

```gleam
import gleam/io

pub fn describe_point(point: #(Int, Int)) -> String {
  case point {
    #(0, 0) -> "origin"
    #(0, y) -> "on y-axis at " <> int.to_string(y)
    #(x, 0) -> "on x-axis at " <> int.to_string(x)
    #(x, y) -> "at (" <> int.to_string(x) <> ", " <> int.to_string(y) <> ")"
  }
}

pub fn main() {
  io.println(describe_point(#(0, 0)))    // "origin"
  io.println(describe_point(#(5, 0)))    // "on x-axis at 5"
  io.println(describe_point(#(3, 4)))    // "at (3, 4)"
}
```

### Pattern Matching on Lists

```gleam
import gleam/io

pub fn describe_list(list: List(Int)) -> String {
  case list {
    [] -> "empty list"
    [x] -> "single element: " <> int.to_string(x)
    [x, y] -> "two elements: " <> int.to_string(x) <> " and " <> int.to_string(y)
    [first, ..rest] -> "starts with " <> int.to_string(first)
  }
}

pub fn main() {
  io.println(describe_list([]))           // "empty list"
  io.println(describe_list([1]))          // "single element: 1"
  io.println(describe_list([1, 2]))       // "two elements: 1 and 2"
  io.println(describe_list([1, 2, 3]))    // "starts with 1"
}
```

### Guard Clauses

Guards add additional conditions to patterns:

```gleam
pub fn categorize_age(age: Int) -> String {
  case age {
    age if age < 0 -> "invalid"
    age if age < 13 -> "child"
    age if age < 20 -> "teenager"
    age if age < 65 -> "adult"
    _ -> "senior"
  }
}
```

---

## Custom Types

Custom types allow you to define your own data structures. They're similar to enums or algebraic data types in other languages.

### Simple Custom Types

```gleam
import gleam/io

// Define a custom type
pub type Season {
  Spring
  Summer
  Autumn
  Winter
}

pub fn season_temperature(season: Season) -> String {
  case season {
    Spring -> "Mild"
    Summer -> "Hot"
    Autumn -> "Cool"
    Winter -> "Cold"
  }
}

pub fn main() {
  let current_season = Summer
  io.println(season_temperature(current_season))  // "Hot"
}
```

### Custom Types with Data

```gleam
import gleam/io

pub type Shape {
  Circle(radius: Float)
  Rectangle(width: Float, height: Float)
  Square(side: Float)
}

pub fn area(shape: Shape) -> Float {
  case shape {
    Circle(radius) -> 3.14159 *. radius *. radius
    Rectangle(width, height) -> width *. height
    Square(side) -> side *. side
  }
}

pub fn main() {
  let circle = Circle(radius: 5.0)
  let rectangle = Rectangle(width: 4.0, height: 6.0)
  let square = Square(side: 3.0)
  
  io.debug(area(circle))      // ~78.54
  io.debug(area(rectangle))   // 24.0
  io.debug(area(square))      // 9.0
}
```

### Option Type

Gleam doesn't have null/nil for values. Instead, it uses the `Option` type:

```gleam
import gleam/io
import gleam/option.{type Option, None, Some}

pub fn find_user(id: Int) -> Option(String) {
  case id {
    1 -> Some("Alice")
    2 -> Some("Bob")
    _ -> None
  }
}

pub fn main() {
  case find_user(1) {
    Some(name) -> io.println("Found: " <> name)
    None -> io.println("User not found")
  }
  
  case find_user(999) {
    Some(name) -> io.println("Found: " <> name)
    None -> io.println("User not found")
  }
}
```

### Working with Option Values

```gleam
import gleam/option.{type Option, None, Some}
import gleam/io

pub fn get_user_email(user_id: Int) -> Option(String) {
  case user_id {
    1 -> Some("alice@example.com")
    2 -> Some("bob@example.com")
    _ -> None
  }
}

pub fn send_email(email: String) -> Nil {
  io.println("Sending email to: " <> email)
}

pub fn main() {
  let user_id = 1
  
  // Pattern match on Option
  case get_user_email(user_id) {
    Some(email) -> send_email(email)
    None -> io.println("No email found")
  }
  
  // Using option.map
  let _ = 
    get_user_email(user_id)
    |> option.map(send_email)
  
  // Unwrap with default
  let email = option.unwrap(get_user_email(user_id), "no-reply@example.com")
  io.println(email)
}
```

### Generic Custom Types

Custom types can be generic (parameterized):

```gleam
import gleam/io

pub type Box(a) {
  Box(value: a)
}

pub fn main() {
  let int_box = Box(value: 42)
  let string_box = Box(value: "Hello")
  let bool_box = Box(value: True)
  
  case int_box {
    Box(value) -> io.debug(value)
  }
  
  case string_box {
    Box(value) -> io.println(value)
  }
}
```

### Type Aliases

You can create aliases for complex types:

```gleam
pub type UserId = Int
pub type Email = String
pub type Username = String

pub type User {
  User(id: UserId, username: Username, email: Email)
}

pub fn create_user(id: Int, username: String, email: String) -> User {
  User(id: id, username: username, email: email)
}
```

---

## Lists and List Operations

Lists are one of the most commonly used data structures in Gleam. They're immutable and linked lists.

### Creating Lists

```gleam
import gleam/io
import gleam/list

pub fn main() {
  // Empty list
  let empty = []
  
  // List of integers
  let numbers = [1, 2, 3, 4, 5]
  
  // List of strings
  let names = ["Alice", "Bob", "Charlie"]
  
  // Prepending to a list (fast operation)
  let new_numbers = [0, ..numbers]  // [0, 1, 2, 3, 4, 5]
  
  io.debug(new_numbers)
}
```

### Common List Operations

```gleam
import gleam/list
import gleam/io

pub fn list_operations() {
  let numbers = [1, 2, 3, 4, 5]
  
  // Length
  let len = list.length(numbers)  // 5
  
  // First element
  let first = list.first(numbers)  // Ok(1)
  
  // Rest of the list
  let rest = list.rest(numbers)    // Ok([2, 3, 4, 5])
  
  // Reverse
  let reversed = list.reverse(numbers)  // [5, 4, 3, 2, 1]
  
  // Append (slow operation)
  let appended = list.append(numbers, [6, 7])  // [1, 2, 3, 4, 5, 6, 7]
  
  // Contains
  let has_3 = list.contains(numbers, 3)  // True
  
  io.debug(len)
  io.debug(reversed)
}
```

### List.map - Transform Elements

```gleam
import gleam/list
import gleam/io

pub fn map_examples() {
  let numbers = [1, 2, 3, 4, 5]
  
  // Double each number
  let doubled = list.map(numbers, fn(x) { x * 2 })
  io.debug(doubled)  // [2, 4, 6, 8, 10]
  
  // Convert to strings
  let strings = list.map(numbers, int.to_string)
  io.debug(strings)  // ["1", "2", "3", "4", "5"]
}
```

### List.filter - Select Elements

```gleam
import gleam/list
import gleam/io

pub fn filter_examples() {
  let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  
  // Keep only even numbers
  let evens = list.filter(numbers, fn(x) { x % 2 == 0 })
  io.debug(evens)  // [2, 4, 6, 8, 10]
  
  // Keep only numbers greater than 5
  let greater_than_5 = list.filter(numbers, fn(x) { x > 5 })
  io.debug(greater_than_5)  // [6, 7, 8, 9, 10]
}
```

### List.fold - Reduce to Single Value

```gleam
import gleam/list
import gleam/io

pub fn fold_examples() {
  let numbers = [1, 2, 3, 4, 5]
  
  // Sum all numbers
  let sum = list.fold(numbers, 0, fn(acc, x) { acc + x })
  io.debug(sum)  // 15
  
  // Product of all numbers
  let product = list.fold(numbers, 1, fn(acc, x) { acc * x })
  io.debug(product)  // 120
  
  // Concatenate strings
  let words = ["Hello", "World", "From", "Gleam"]
  let sentence = list.fold(words, "", fn(acc, word) { acc <> " " <> word })
  io.debug(sentence)  // " Hello World From Gleam"
}
```

### List Comprehensions (Alternative Style)

Gleam uses combinators instead of list comprehensions:

```gleam
import gleam/list
import gleam/io

pub fn list_combinators() {
  let numbers = [1, 2, 3, 4, 5]
  
  // Get squares of even numbers
  let result = 
    numbers
    |> list.filter(fn(x) { x % 2 == 0 })
    |> list.map(fn(x) { x * x })
  
  io.debug(result)  // [4, 16]
}
```

### Recursive List Processing

```gleam
import gleam/io

pub fn sum_list(numbers: List(Int)) -> Int {
  case numbers {
    [] -> 0
    [first, ..rest] -> first + sum_list(rest)
  }
}

pub fn list_length(list: List(a)) -> Int {
  case list {
    [] -> 0
    [_, ..rest] -> 1 + list_length(rest)
  }
}

pub fn main() {
  let numbers = [1, 2, 3, 4, 5]
  io.debug(sum_list(numbers))      // 15
  io.debug(list_length(numbers))   // 5
}
```

---

## Tuples and Records

### Tuples

Tuples are fixed-size collections of values that can have different types:

```gleam
import gleam/io

pub fn tuple_examples() {
  // 2-tuple (pair)
  let point = #(3, 4)
  
  // 3-tuple
  let person = #("Alice", 30, "alice@example.com")
  
  // Accessing tuple elements via pattern matching
  let #(x, y) = point
  io.debug(x)  // 3
  io.debug(y)  // 4
  
  let #(name, age, email) = person
  io.println(name)  // "Alice"
  
  // Nested tuples
  let nested = #(#(1, 2), #(3, 4))
  let #(#(a, b), #(c, d)) = nested
}
```

### Using Tuples in Functions

```gleam
import gleam/io

pub fn swap(tuple: #(a, b)) -> #(b, a) {
  let #(first, second) = tuple
  #(second, first)
}

pub fn add_points(p1: #(Int, Int), p2: #(Int, Int)) -> #(Int, Int) {
  let #(x1, y1) = p1
  let #(x2, y2) = p2
  #(x1 + x2, y1 + y2)
}

pub fn main() {
  let original = #("Hello", 42)
  let swapped = swap(original)
  io.debug(swapped)  // #(42, "Hello")
  
  let point1 = #(1, 2)
  let point2 = #(3, 4)
  let sum = add_points(point1, point2)
  io.debug(sum)  // #(4, 6)
}
```

### Records (Named Fields)

While tuples have positional fields, custom types provide named fields:

```gleam
import gleam/io

pub type Person {
  Person(name: String, age: Int, email: String)
}

pub fn create_person(name: String, age: Int, email: String) -> Person {
  Person(name: name, age: age, email: email)
}

pub fn get_older(person: Person, years: Int) -> Person {
  Person(..person, age: person.age + years)
}

pub fn main() {
  let alice = create_person("Alice", 30, "alice@example.com")
  
  // Access fields
  io.println(alice.name)   // "Alice"
  io.debug(alice.age)      // 30
  
  // Update using spread syntax
  let older_alice = get_older(alice, 5)
  io.debug(older_alice.age)  // 35
  
  // Update multiple fields
  let updated = Person(..alice, name: "Alice Smith", email: "new@example.com")
  io.println(updated.name)  // "Alice Smith"
}
```

---

## Error Handling with Result

Gleam doesn't use exceptions. Instead, it uses the `Result` type for operations that can fail.

### The Result Type

```gleam
pub type Result(value, error) {
  Ok(value)
  Error(error)
}
```

### Basic Result Usage

```gleam
import gleam/io
import gleam/int
import gleam/result

pub fn divide(a: Int, b: Int) -> Result(Int, String) {
  case b {
    0 -> Error("Cannot divide by zero")
    _ -> Ok(a / b)
  }
}

pub fn main() {
  case divide(10, 2) {
    Ok(result) -> io.debug(result)  // 5
    Error(message) -> io.println("Error: " <> message)
  }
  
  case divide(10, 0) {
    Ok(result) -> io.debug(result)
    Error(message) -> io.println("Error: " <> message)  // "Error: Cannot divide by zero"
  }
}
```

### Parsing with Result

```gleam
import gleam/io
import gleam/int

pub fn parse_age(input: String) -> Result(Int, String) {
  case int.parse(input) {
    Ok(age) if age >= 0 && age <= 150 -> Ok(age)
    Ok(_) -> Error("Age out of valid range")
    Error(_) -> Error("Invalid number format")
  }
}

pub fn main() {
  case parse_age("25") {
    Ok(age) -> io.println("Valid age: " <> int.to_string(age))
    Error(msg) -> io.println("Error: " <> msg)
  }
  
  case parse_age("999") {
    Ok(age) -> io.println("Valid age: " <> int.to_string(age))
    Error(msg) -> io.println("Error: " <> msg)  // "Error: Age out of valid range"
  }
}
```

### Chaining Results with try

The `use` keyword (formerly `try`) makes working with multiple Results cleaner:

```gleam
import gleam/io
import gleam/int
import gleam/result

pub fn parse_int(s: String) -> Result(Int, String) {
  case int.parse(s) {
    Ok(n) -> Ok(n)
    Error(_) -> Error("Invalid integer: " <> s)
  }
}

pub fn add_parsed_numbers(a: String, b: String) -> Result(Int, String) {
  use num_a <- result.try(parse_int(a))
  use num_b <- result.try(parse_int(b))
  Ok(num_a + num_b)
}

pub fn main() {
  case add_parsed_numbers("10", "20") {
    Ok(sum) -> io.debug(sum)  // 30
    Error(msg) -> io.println(msg)
  }
  
  case add_parsed_numbers("10", "abc") {
    Ok(sum) -> io.debug(sum)
    Error(msg) -> io.println(msg)  // "Invalid integer: abc"
  }
}
```

### Result Helper Functions

```gleam
import gleam/result
import gleam/io

pub fn result_helpers() {
  let ok_value: Result(Int, String) = Ok(42)
  let error_value: Result(Int, String) = Error("Something went wrong")
  
  // Unwrap with default
  let value1 = result.unwrap(ok_value, 0)  // 42
  let value2 = result.unwrap(error_value, 0)  // 0
  
  // Map over Ok values
  let doubled = result.map(ok_value, fn(x) { x * 2 })  // Ok(84)
  
  // Map over Error values
  let new_error = result.map_error(error_value, fn(e) { "Error: " <> e })
  
  io.debug(value1)
  io.debug(doubled)
}
```

### Custom Error Types

```gleam
import gleam/io

pub type UserError {
  NotFound
  InvalidEmail
  AlreadyExists
}

pub fn find_user(email: String) -> Result(String, UserError) {
  case email {
    "alice@example.com" -> Ok("Alice")
    "bob@example.com" -> Ok("Bob")
    "" -> Error(InvalidEmail)
    _ -> Error(NotFound)
  }
}

pub fn error_message(error: UserError) -> String {
  case error {
    NotFound -> "User not found"
    InvalidEmail -> "Invalid email address"
    AlreadyExists -> "User already exists"
  }
}

pub fn main() {
  case find_user("alice@example.com") {
    Ok(name) -> io.println("Found: " <> name)
    Error(err) -> io.println(error_message(err))
  }
  
  case find_user("unknown@example.com") {
    Ok(name) -> io.println("Found: " <> name)
    Error(err) -> io.println(error_message(err))  // "User not found"
  }
}
```

---

## Modules and Imports

### Creating Modules

Each `.gleam` file is a module. The module name is derived from the file path.

**File: `src/math/calculator.gleam`**

```gleam
// This module is named: math/calculator

pub fn add(a: Int, b: Int) -> Int {
  a + b
}

pub fn subtract(a: Int, b: Int) -> Int {
  a - b
}

// Private function (not exported)
fn internal_multiply(a: Int, b: Int) -> Int {
  a * b
}

pub fn multiply(a: Int, b: Int) -> Int {
  internal_multiply(a, b)
}
```

### Importing Modules

**File: `src/main.gleam`**

```gleam
import gleam/io
import math/calculator

pub fn main() {
  let sum = calculator.add(5, 3)
  io.debug(sum)  // 8
  
  let difference = calculator.subtract(10, 4)
  io.debug(difference)  // 6
}
```

### Import with Alias

```gleam
import gleam/io
import math/calculator as calc

pub fn main() {
  let product = calc.multiply(4, 5)
  io.debug(product)  // 20
}
```

### Unqualified Imports

```gleam
import gleam/io
import math/calculator.{add, subtract}

pub fn main() {
  // Can use add and subtract directly
  let sum = add(5, 3)
  let diff = subtract(10, 4)
  
  io.debug(sum)
  io.debug(diff)
}
```

### Importing Types

```gleam
import gleam/option.{type Option, None, Some}

pub fn find_item(id: Int) -> Option(String) {
  case id {
    1 -> Some("Item 1")
    _ -> None
  }
}
```

### Module Structure Best Practices

**File: `src/user/types.gleam`**

```gleam
pub type User {
  User(id: Int, name: String, email: String)
}

pub type UserError {
  NotFound
  InvalidInput
  DatabaseError
}
```

**File: `src/user/service.gleam`**

```gleam
import user/types.{type User, type UserError, User, NotFound, InvalidInput}
import gleam/io

pub fn create_user(name: String, email: String) -> Result(User, UserError) {
  case name, email {
    "", _ -> Error(InvalidInput)
    _, "" -> Error(InvalidInput)
    _, _ -> Ok(User(id: 1, name: name, email: email))
  }
}

pub fn find_user(id: Int) -> Result(User, UserError) {
  case id {
    1 -> Ok(User(id: 1, name: "Alice", email: "alice@example.com"))
    _ -> Error(NotFound)
  }
}
```

---

## Working with Strings

### String Module Functions

```gleam
import gleam/string
import gleam/io

pub fn string_operations() {
  let text = "Hello, Gleam!"
  
  // Length
  let len = string.length(text)  // 13
  
  // Uppercase and lowercase
  let upper = string.uppercase(text)  // "HELLO, GLEAM!"
  let lower = string.lowercase(text)  // "hello, gleam!"
  
  // Trim whitespace
  let padded = "  hello  "
  let trimmed = string.trim(padded)  // "hello"
  
  // Split
  let words = string.split("one,two,three", ",")  // ["one", "two", "three"]
  
  // Join
  let joined = string.join(["a", "b", "c"], "-")  // "a-b-c"
  
  // Contains
  let has_gleam = string.contains(text, "Gleam")  // True
  
  // Starts with / Ends with
  let starts = string.starts_with(text, "Hello")  // True
  let ends = string.ends_with(text, "!")  // True
  
  io.debug(upper)
  io.debug(words)
}
```

### String Manipulation

```gleam
import gleam/string
import gleam/io

pub fn string_manipulation() {
  let name = "alice"
  
  // Capitalize first letter
  let capitalized = string.capitalise(name)  // "Alice"
  
  // Repeat
  let repeated = string.repeat("ha", 3)  // "hahaha"
  
  // Reverse
  let reversed = string.reverse("hello")  // "olleh"
  
  // Replace
  let replaced = string.replace("hello world", "world", "Gleam")  // "hello Gleam"
  
  // Slice (substring)
  let text = "Gleam is great"
  let sliced = string.slice(text, 0, 5)  // "Gleam"
  
  io.println(capitalized)
  io.println(replaced)
}
```

### Working with String Lists

```gleam
import gleam/string
import gleam/list
import gleam/io

pub fn string_list_operations() {
  let sentence = "The quick brown fox jumps"
  
  // Split into words and process
  let words = string.split(sentence, " ")
  
  // Make all words uppercase
  let uppercase_words = list.map(words, string.uppercase)
  
  // Join back together
  let result = string.join(uppercase_words, " ")
  
  io.println(result)  // "THE QUICK BROWN FOX JUMPS"
  
  // Count words
  let word_count = list.length(words)
  io.debug(word_count)  // 5
}
```

---

## Higher-Order Functions

Higher-order functions are functions that take other functions as arguments or return functions.

### Functions as Arguments

```gleam
import gleam/io

pub fn apply_twice(f: fn(Int) -> Int, x: Int) -> Int {
  f(f(x))
}

pub fn double(x: Int) -> Int {
  x * 2
}

pub fn increment(x: Int) -> Int {
  x + 1
}

pub fn main() {
  let result1 = apply_twice(double, 3)  // double(double(3)) = 12
  let result2 = apply_twice(increment, 5)  // increment(increment(5)) = 7
  
  io.debug(result1)
  io.debug(result2)
}
```

### Returning Functions

```gleam
import gleam/io

pub fn make_adder(x: Int) -> fn(Int) -> Int {
  fn(y) { x + y }
}

pub fn main() {
  let add_5 = make_adder(5)
  let add_10 = make_adder(10)
  
  io.debug(add_5(3))   // 8
  io.debug(add_10(3))  // 13
}
```

### Common Higher-Order Functions

```gleam
import gleam/list
import gleam/io

pub fn higher_order_examples() {
  let numbers = [1, 2, 3, 4, 5]
  
  // Map - transform each element
  let doubled = list.map(numbers, fn(x) { x * 2 })
  
  // Filter - keep elements matching predicate
  let evens = list.filter(numbers, fn(x) { x % 2 == 0 })
  
  // Fold - reduce to single value
  let sum = list.fold(numbers, 0, fn(acc, x) { acc + x })
  
  // Find - get first matching element
  let first_even = list.find(numbers, fn(x) { x % 2 == 0 })
  
  // All - check if all elements match
  let all_positive = list.all(numbers, fn(x) { x > 0 })
  
  // Any - check if any element matches
  let has_even = list.any(numbers, fn(x) { x % 2 == 0 })
  
  io.debug(doubled)  // [2, 4, 6, 8, 10]
  io.debug(evens)    // [2, 4]
  io.debug(sum)      // 15
}
```

### Function Composition

```gleam
import gleam/io

pub fn compose(
  f: fn(b) -> c,
  g: fn(a) -> b,
) -> fn(a) -> c {
  fn(x) { f(g(x)) }
}

pub fn double(x: Int) -> Int {
  x * 2
}

pub fn add_one(x: Int) -> Int {
  x + 1
}

pub fn main() {
  // Compose: first add_one, then double
  let double_after_increment = compose(double, add_one)
  
  io.debug(double_after_increment(5))  // double(add_one(5)) = double(6) = 12
}
```

### Practical Example: Data Processing Pipeline

```gleam
import gleam/list
import gleam/string
import gleam/io
import gleam/int

pub type Product {
  Product(name: String, price: Int, in_stock: Bool)
}

pub fn process_products(products: List(Product)) -> String {
  products
  |> list.filter(fn(p) { p.in_stock })
  |> list.filter(fn(p) { p.price < 100 })
  |> list.map(fn(p) { p.name })
  |> list.map(string.uppercase)
  |> string.join(", ")
}

pub fn main() {
  let products = [
    Product("laptop", 1200, True),
    Product("mouse", 25, True),
    Product("keyboard", 75, True),
    Product("monitor", 300, False),
    Product("webcam", 50, True),
  ]
  
  let result = process_products(products)
  io.println(result)  // "MOUSE, KEYBOARD, WEBCAM"
}
```

---

## Type System Deep Dive

### Type Inference

Gleam has powerful type inference, so you often don't need to write type annotations:

```gleam
pub fn main() {
  // Type is inferred as Int
  let x = 42
  
  // Type is inferred as String
  let name = "Alice"
  
  // Type is inferred as List(Int)
  let numbers = [1, 2, 3]
  
  // Type is inferred as fn(Int) -> Int
  let double = fn(x) { x * 2 }
}
```

### Explicit Type Annotations

You can add type annotations for clarity or to help the compiler:

```gleam
pub fn greet(name: String) -> String {
  "Hello, " <> name
}

pub fn calculate(x: Int, y: Int) -> Int {
  x + y
}

pub fn main() {
  let age: Int = 25
  let names: List(String) = ["Alice", "Bob"]
  let point: #(Int, Int) = #(3, 4)
}
```

### Generic Functions

Functions can be generic over types:

```gleam
import gleam/io

// Generic identity function
pub fn identity(x: a) -> a {
  x
}

// Generic function with multiple type parameters
pub fn pair(first: a, second: b) -> #(a, b) {
  #(first, second)
}

// Generic list functions
pub fn first(list: List(a)) -> Result(a, Nil) {
  case list {
    [x, .._] -> Ok(x)
    [] -> Error(Nil)
  }
}

pub fn main() {
  io.debug(identity(42))        // 42
  io.debug(identity("hello"))   // "hello"
  
  io.debug(pair(1, "one"))      // #(1, "one")
  io.debug(pair(True, 3.14))    // #(True, 3.14)
}
```

### Type Constraints

While Gleam doesn't have type classes like Haskell, you can constrain types using custom types:

```gleam
pub type Comparable(a) {
  Comparable(compare: fn(a, a) -> Order)
}

pub type Order {
  LessThan
  Equal
  GreaterThan
}

pub fn max(comparable: Comparable(a), x: a, y: a) -> a {
  case comparable.compare(x, y) {
    LessThan -> y
    Equal -> x
    GreaterThan -> x
  }
}

pub fn int_comparable() -> Comparable(Int) {
  Comparable(compare: fn(a, b) {
    case a < b, a == b {
      True, _ -> LessThan
      _, True -> Equal
      _, _ -> GreaterThan
    }
  })
}
```

### Opaque Types

Opaque types hide their internal structure from other modules:

**File: `src/user_id.gleam`**

```gleam
// The internal structure is hidden
pub opaque type UserId {
  UserId(value: Int)
}

pub fn new(id: Int) -> Result(UserId, String) {
  case id > 0 {
    True -> Ok(UserId(id))
    False -> Error("User ID must be positive")
  }
}

pub fn to_int(user_id: UserId) -> Int {
  user_id.value
}
```

**File: `src/main.gleam`**

```gleam
import user_id
import gleam/io

pub fn main() {
  // Can't create UserId directly - must use constructor
  // let id = user_id.UserId(123)  // ERROR! Constructor is private
  
  // Must use the public API
  case user_id.new(123) {
    Ok(id) -> {
      let value = user_id.to_int(id)
      io.debug(value)
    }
    Error(msg) -> io.println(msg)
  }
}
```

---

## Building a Real Project

Let's build a simple TODO list application that demonstrates many Gleam concepts.

### Project Structure

```bash
gleam new todo_app
cd todo_app
```

### Define Types

**File: `src/todo/types.gleam`**

```gleam
pub type TodoId = Int

pub type Todo {
  Todo(id: TodoId, title: String, completed: Bool)
}

pub type TodoError {
  NotFound
  EmptyTitle
  AlreadyExists
}
```

### Implement Todo Service

**File: `src/todo/service.gleam`**

```gleam
import todo/types.{type Todo, type TodoError, type TodoId, Todo, NotFound, EmptyTitle, AlreadyExists}
import gleam/list
import gleam/string

pub type TodoList {
  TodoList(todos: List(Todo), next_id: Int)
}

pub fn new() -> TodoList {
  TodoList(todos: [], next_id: 1)
}

pub fn add_todo(list: TodoList, title: String) -> Result(#(TodoList, Todo), TodoError) {
  case string.trim(title) {
    "" -> Error(EmptyTitle)
    trimmed_title -> {
      let todo = Todo(id: list.next_id, title: trimmed_title, completed: False)
      let new_list = TodoList(
        todos: [todo, ..list.todos],
        next_id: list.next_id + 1
      )
      Ok(#(new_list, todo))
    }
  }
}

pub fn complete_todo(list: TodoList, id: TodoId) -> Result(TodoList, TodoError) {
  case find_and_update(list.todos, id, fn(todo) { Todo(..todo, completed: True) }) {
    Ok(updated_todos) -> Ok(TodoList(..list, todos: updated_todos))
    Error(_) -> Error(NotFound)
  }
}

pub fn delete_todo(list: TodoList, id: TodoId) -> Result(TodoList, TodoError) {
  let filtered = list.filter(list.todos, fn(todo) { todo.id != id })
  case list.length(filtered) == list.length(list.todos) {
    True -> Error(NotFound)
    False -> Ok(TodoList(..list, todos: filtered))
  }
}

pub fn get_all(list: TodoList) -> List(Todo) {
  list.reverse(list.todos)
}

pub fn get_active(list: TodoList) -> List(Todo) {
  list.todos
  |> list.filter(fn(todo) { !todo.completed })
  |> list.reverse
}

pub fn get_completed(list: TodoList) -> List(Todo) {
  list.todos
  |> list.filter(fn(todo) { todo.completed })
  |> list.reverse
}

fn find_and_update(
  todos: List(Todo),
  id: TodoId,
  updater: fn(Todo) -> Todo
) -> Result(List(Todo), Nil) {
  case todos {
    [] -> Error(Nil)
    [todo, ..rest] if todo.id == id -> Ok([updater(todo), ..rest])
    [todo, ..rest] -> {
      case find_and_update(rest, id, updater) {
        Ok(updated_rest) -> Ok([todo, ..updated_rest])
        Error(_) -> Error(Nil)
      }
    }
  }
}
```

### Add Display Functions

**File: `src/todo/display.gleam`**

```gleam
import todo/types.{type Todo}
import gleam/string
import gleam/list
import gleam/int

pub fn format_todo(todo: Todo) -> String {
  let status = case todo.completed {
    True -> "[✓]"
    False -> "[ ]"
  }
  status <> " " <> int.to_string(todo.id) <> ". " <> todo.title
}

pub fn format_todo_list(todos: List(Todo)) -> String {
  case todos {
    [] -> "No todos"
    _ -> {
      todos
      |> list.map(format_todo)
      |> string.join("\n")
    }
  }
}
```

### Main Application

**File: `src/todo_app.gleam`**

```gleam
import gleam/io
import todo/service
import todo/display
import todo/types

pub fn main() {
  io.println("=== Todo List Application ===\n")
  
  // Create new todo list
  let list = service.new()
  
  // Add some todos
  let assert Ok(#(list, todo1)) = service.add_todo(list, "Learn Gleam")
  io.println("Added: " <> display.format_todo(todo1))
  
  let assert Ok(#(list, todo2)) = service.add_todo(list, "Build a project")
  io.println("Added: " <> display.format_todo(todo2))
  
  let assert Ok(#(list, todo3)) = service.add_todo(list, "Write tests")
  io.println("Added: " <> display.format_todo(todo3))
  
  // Show all todos
  io.println("\n--- All Todos ---")
  io.println(display.format_todo_list(service.get_all(list)))
  
  // Complete a todo
  let assert Ok(list) = service.complete_todo(list, todo1.id)
  io.println("\n--- After completing first todo ---")
  io.println(display.format_todo_list(service.get_all(list)))
  
  // Show only active todos
  io.println("\n--- Active Todos ---")
  io.println(display.format_todo_list(service.get_active(list)))
  
  // Show only completed todos
  io.println("\n--- Completed Todos ---")
  io.println(display.format_todo_list(service.get_completed(list)))
  
  // Delete a todo
  let assert Ok(list) = service.delete_todo(list, todo2.id)
  io.println("\n--- After deleting second todo ---")
  io.println(display.format_todo_list(service.get_all(list)))
  
  // Try to add empty todo
  case service.add_todo(list, "   ") {
    Ok(_) -> io.println("Should not happen")
    Error(types.EmptyTitle) -> io.println("\nCannot add empty todo!")
    Error(_) -> io.println("Other error")
  }
}
```

### Running the Application

```bash
gleam run
```

**Output:**

```
=== Todo List Application ===

Added: [ ] 1. Learn Gleam
Added: [ ] 2. Build a project
Added: [ ] 3. Write tests

--- All Todos ---
[ ] 1. Learn Gleam
[ ] 2. Build a project
[ ] 3. Write tests

--- After completing first todo ---
[✓] 1. Learn Gleam
[ ] 2. Build a project
[ ] 3. Write tests

--- Active Todos ---
[ ] 2. Build a project
[ ] 3. Write tests

--- Completed Todos ---
[✓] 1. Learn Gleam

--- After deleting second todo ---
[✓] 1. Learn Gleam
[ ] 3. Write tests

Cannot add empty todo!
```

---

## Best Practices

### 1. Use Descriptive Names

```gleam
// Good
pub fn calculate_total_price(items: List(Item)) -> Float { ... }

// Not so good
pub fn calc(x: List(Item)) -> Float { ... }
```

### 2. Prefer Pattern Matching Over Boolean Checks

```gleam
// Good
pub fn describe_result(result: Result(Int, String)) -> String {
  case result {
    Ok(value) -> "Success: " <> int.to_string(value)
    Error(message) -> "Error: " <> message
  }
}

// Less idiomatic
pub fn describe_result_bad(result: Result(Int, String)) -> String {
  case result.is_ok(result) {
    True -> "Success"
    False -> "Error"
  }
}
```

### 3. Use the Pipe Operator for Readability

```gleam
// Good - clear flow of data
pub fn process_names(names: List(String)) -> String {
  names
  |> list.map(string.trim)
  |> list.filter(fn(name) { string.length(name) > 0 })
  |> list.map(string.capitalise)
  |> string.join(", ")
}

// Harder to read
pub fn process_names_nested(names: List(String)) -> String {
  string.join(
    list.map(
      list.filter(
        list.map(names, string.trim),
        fn(name) { string.length(name) > 0 }
      ),
      string.capitalise
    ),
    ", "
  )
}
```

### 4. Keep Functions Small and Focused

```gleam
// Good - single responsibility
pub fn validate_email(email: String) -> Bool {
  string.contains(email, "@") && string.contains(email, ".")
}

pub fn validate_age(age: Int) -> Bool {
  age >= 0 && age <= 150
}

pub fn validate_user(email: String, age: Int) -> Result(Nil, String) {
  case validate_email(email), validate_age(age) {
    False, _ -> Error("Invalid email")
    _, False -> Error("Invalid age")
    True, True -> Ok(Nil)
  }
}
```

### 5. Use Custom Types for Domain Modeling

```gleam
// Good - types represent domain concepts
pub type EmailAddress {
  EmailAddress(value: String)
}

pub type Age {
  Age(years: Int)
}

pub type User {
  User(email: EmailAddress, age: Age)
}

// Less type-safe
pub type UserBad {
  UserBad(email: String, age: Int)
}
```

### 6. Handle All Cases in Pattern Matching

```gleam
// Good - all cases handled
pub fn describe_option(option: Option(Int)) -> String {
  case option {
    Some(value) -> "Value: " <> int.to_string(value)
    None -> "No value"
  }
}

// Compiler will warn if you miss a case
```

### 7. Use Result for Error Handling

```gleam
// Good - explicit error handling
pub fn divide(a: Float, b: Float) -> Result(Float, String) {
  case b {
    0.0 -> Error("Division by zero")
    _ -> Ok(a /. b)
  }
}

// Don't use panic! unless truly impossible
pub fn divide_bad(a: Float, b: Float) -> Float {
  case b {
    0.0 -> panic as "Division by zero"
    _ -> a /. b
  }
}
```

### 8. Document Public APIs

```gleam
/// Calculates the factorial of a non-negative integer.
/// Returns Error if the input is negative.
///
/// ## Examples
///
/// ```gleam
/// factorial(5)
/// // -> Ok(120)
///
/// factorial(-1)
/// // -> Error("Input must be non-negative")
/// ```
pub fn factorial(n: Int) -> Result(Int, String) {
  case n {
    n if n < 0 -> Error("Input must be non-negative")
    0 -> Ok(1)
    n -> {
      use prev <- result.try(factorial(n - 1))
      Ok(n * prev)
    }
  }
}
```

### 9. Use Opaque Types for Encapsulation

```gleam
/// Represents a validated email address
pub opaque type Email {
  Email(value: String)
}

/// Creates a new Email if the string is valid
pub fn new(email: String) -> Result(Email, String) {
  case string.contains(email, "@") && string.contains(email, ".") {
    True -> Ok(Email(email))
    False -> Error("Invalid email format")
  }
}

/// Gets the string value of an Email
pub fn to_string(email: Email) -> String {
  email.value
}
```

### 10. Test Your Code

**File: `test/todo_app_test.gleam`**

```gleam
import gleeunit
import gleeunit/should
import todo/service
import todo/types

pub fn main() {
  gleeunit.main()
}

pub fn add_todo_test() {
  let list = service.new()
  
  let assert Ok(#(list, todo)) = service.add_todo(list, "Test todo")
  
  todo.title
  |> should.equal("Test todo")
  
  todo.completed
  |> should.be_false()
}

pub fn add_empty_todo_test() {
  let list = service.new()
  
  service.add_todo(list, "   ")
  |> should.be_error()
}

pub fn complete_todo_test() {
  let list = service.new()
  let assert Ok(#(list, todo)) = service.add_todo(list, "Test")
  
  let assert Ok(updated_list) = service.complete_todo(list, todo.id)
  
  let todos = service.get_completed(updated_list)
  list.length(todos)
  |> should.equal(1)
}
```

Run tests:

```bash
gleam test
```

---

## Conclusion

Congratulations! You've completed this comprehensive Gleam tutorial. You've learned:

- ✅ Basic syntax and data types
- ✅ Functions and the pipe operator
- ✅ Pattern matching
- ✅ Custom types and type system
- ✅ Lists and common operations
- ✅ Error handling with Result
- ✅ Modules and imports
- ✅ Higher-order functions
- ✅ Building real applications
- ✅ Best practices

### Next Steps

1. **Explore the Standard Library**: Check out [hexdocs.pm/gleam_stdlib](https://hexdocs.pm/gleam_stdlib/)
2. **Build More Projects**: Practice by building CLI tools, web servers, or data processors
3. **Learn Web Development**: Explore [Wisp](https://github.com/gleam-lang/wisp) for web applications
4. **Join the Community**: Visit [gleam.run](https://gleam.run) and join the Discord
5. **Read Official Documentation**: [gleam.run/documentation](https://gleam.run/documentation)

Happy coding with Gleam! 🌟