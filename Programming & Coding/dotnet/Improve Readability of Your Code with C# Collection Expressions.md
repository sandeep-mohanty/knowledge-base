# Improve Readability of Your Code with C# Collection Expressions

C# 12 introduced one interesting feature called **collection expressions**. Collection expressions allow writing more concise and readable code when working with collections.

[](#collection-initializers)

## Collection Initializers

Collection expressions provide a simple and consistent syntax across many different collection types. When initializing a collection with a collection expression, the compiler generates code that is functionally equivalent to using a collection initializer.

Let's explore the usual ways to initialize a list:

```csharp
// Using var
var list1 = new List<int> { 1, 2, 3, 4, 5 };

// Or using new()
List<int> list2 = new() { 1, 2, 3, 4, 5 };
```

Collection expressions provide a new syntax to initialize collections:

```csharp
List<int> list = [ 1, 2, 3, 4, 5 ];
```

You can also replace the usual array initialization:

```csharp
var array1 = new int[5] { 1, 2, 3, 4, 5 };
var array2 = new int[] { 1, 2, 3, 4, 5 };
var array3 = new[] { 1, 2, 3, 4, 5 };
```

With a new syntax:

```csharp
int[] array = [ 1, 2, 3, 4, 5 ];
```

When using a new syntax, you need to specify a concrete type of collection, as it can't be inferred directly from the collection initialization.

You can use this new collection expression initialization for collections of any type. Let's explore a few examples:

```csharp
char[] letters = [ 'a', 'b', 'c', 'd' ];
List<string> names = [ "Anton", "Bill", "John" ];
List<Person> persons = [
    new("Anton", 30),
    new("Bill", 25),
    new("John", 20)
];
```

Personally, I like this new syntax, it makes code more concise and more readable for me.

[](#initializing-empty-collections)

## Initializing Empty Collections

You can use a new syntax to initialize empty collections in a new more concise and readable way.

Here is the most common approach to initialize an empty array and list:

```csharp
var emptyArray = new int[0] {};
var emptyList = new List<int>();
```

When using this approach, you allocate memory to create an empty collection. So instead, you can use the following initializers that don't allocate any memory at all:

```csharp
var emptyArray = Array.Empty<int>();
var emptyList = Enumerable.Empty<int>();
```

But you can replace all these with a new empty collection expression initializer:

```csharp
int[] emptyArray = [];
List<int> emptyList = [];
```

This approach is more concise and readable and moreover, the compiler translates it to using `Array.Empty<T>` and `Enumerable.Empty<T>` under the hood, so you don't need to worry about memory.

[](#using-a-spread-element)

## Using a Spread Element

The spread element (**..** two dots) allows you to include the elements of one collection into another collection concisely. This feature is particularly useful when you want to combine or merge collections.

Let's say you have two arrays of numbers, and you want to combine them into a single array:

```csharp
int[] oneTwoThree = [1, 2, 3];
int[] fourFiveSix = [4, 5, 6];
int[] allNumbers = [..oneTwoThree, 50, 60, ..fourFiveSix];

Console.WriteLine(string.Join(", ", allNumbers));
Console.WriteLine($"Length: {allNumbers.Length}");
```

This will output the following result:

```
1, 2, 3, 50, 60, 4, 5, 6
Length: 8
```

It doesn't end up with numbers, you can combine any objects you want, for example, strings:

```csharp
string[] greetings = ["Hello", "Hi"];
string[] farewells = ["Goodbye", "See you"];
string[] allMessages = [..greetings, "How are you?", ..farewells];

Console.WriteLine(string.Join(", ", allMessages));
Console.WriteLine($"Length: {allMessages.Length}");
```

That outputs:

```
Hello, Hi, How are you?, Goodbye, See you
Length: 5
```

And objects:

```csharp
Person[] groupA = [
    new Person("John", 20),
    new Person("Jane", 22)
];

Person[] groupB = [
    new Person("Alice", 25),
    new Person("Bob", 27)
];

Person[] allPeople = [..groupA, new Person("Charlie", 30), ..groupB];

foreach (var person in allPeople)
{
    Console.WriteLine(person);
}
```

That outputs:

```
Person { Name = John, Age = 20 }
Person { Name = Jane, Age = 22 }
Person { Name = Charlie, Age = 30 }
Person { Name = Alice, Age = 25 }
Person { Name = Bob, Age = 27 }
```

This approach is really nice to read and it is easier to write such code than manually creating a list that combines all the values.

This reminds me of a spread operator in JavaScript and TypeScript, but in C# it's not that powerful yet. And in C# it's not an operator but an element. Hope it will be further improved in the future and will allow us to combine objects with a spread syntax.

[](#summary)

## Summary

**Collection expressions** are a great feature introduced in C# 12. You can use collection initializers and spread element to write more concise and readable code when working with collections.

