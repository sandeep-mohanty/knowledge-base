# Can You Solve These Java Stream Problems Without Googling?

Most developers stop at `filter()`, `map()`, and `collect()`.
But in real interviews — especially in product-based companies — that’s not enough.
What truly separates a mid-level developer from a senior engineer is the ability to:
* Solve complex data transformation problems
* Write clean, optimized stream pipelines
* Understand when NOT to use Streams

In this article, we go beyond basics and dive into **advanced Java 8 Stream patterns** that are:
* Frequently asked in interviews
* Used in real-world backend systems
* Designed to test your thinking, not just syntax

From finding the **top N salaries dynamically** to **multi-level grouping and custom sorting**, these problems will push you to think like a senior developer.

---

### 1. Top N Highest Salaries (Dynamic, Not Hardcoded)
```java
int N = 3;

List<Integer> topSalaries =
    employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .limit(N)
        .collect(Collectors.toList());
```
**Explanation:**
* `distinct()` → removes duplicate salaries
* `sorted(reverse)` → highest first
* `limit(N)` → dynamic top values

**Interview Insight:**
Better than `skip()` because it’s **dynamic and scalable**

---

### 2. Find First Non-Repeated Character in String
```java
String input = "swiss";

Optional<Character> result =
    input.chars()
        .mapToObj(c -> (char) c)
        .collect(Collectors.groupingBy(
            Function.identity(),
            LinkedHashMap::new,
            Collectors.counting()
        ))
        .entrySet()
        .stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst();
```
**Explanation:**
* `LinkedHashMap` → maintains order
* `groupingBy + counting` → frequency
* `findFirst()` → first unique

---

### 3. Merge Two Lists Without Duplicates
```java
List<Integer> merged =
    Stream.concat(list1.stream(), list2.stream())
          .distinct()
          .collect(Collectors.toList());
```
**Why important:**
Common in microservices (merge API results)

---

### 4. Find All Anagrams Grouping
```java
Map<String, List<String>> anagrams =
    words.stream()
        .collect(Collectors.groupingBy(word -> {
            char[] arr = word.toCharArray();
            Arrays.sort(arr);
            return new String(arr);
        }));
```
**Explanation:**
**Key idea:** sorted word is same for anagrams
**Example:** “eat”, “tea” → “aet”

---

### 5. Sum of Digits of All Numbers in List
```java
int sum =
    list.stream()
        .flatMap(n -> String.valueOf(n).chars().mapToObj(c -> c - '0'))
        .reduce(0, Integer::sum);
```
**Explanation:**
* Convert number → string → digits
* `flatMap` → flatten digits

---

### 6. Find Longest Increasing Subsequence (Simplified Stream Logic)
Streams alone can’t fully solve LIS efficiently, but we can simulate:

```java
List<Integer> sorted =
    list.stream()
        .sorted()
        .collect(Collectors.toList());
```
**Interview Insight:**
Say this:
“Streams are not ideal for complex DP problems like LIS — better use loops or recursion.”
This shows maturity

---

### 7. Group Employees by Dept → Average Salary
```java
Map<String, Double> avgSalary =
    employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.averagingInt(Employee::getSalary)
        ));
```
**Real-world:**
Used in analytics dashboards

---

### 8. Find Kth Smallest Element
```java
int k = 3;

Optional<Integer> kth =
    list.stream()
        .sorted()
        .skip(k - 1)
        .findFirst();
```
**Explanation:**
* Sorting + skipping
* Clean but O(n log n)

---

### 9. Detect Palindrome Strings in List
```java
List<String> palindromes =
    list.stream()
        .filter(s -> s.equals(new StringBuilder(s).reverse().toString()))
        .collect(Collectors.toList());
```

---

### 10. Multi-Level Grouping (Dept → Role)
```java
Map<String, Map<String, List<Employee>>> result =
    employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.groupingBy(Employee::getRole)
        ));
```
**Output:**
* IT → { Developer → [...], Tester → [...] }
* HR → { Manager → [...] }

---

### ULTRA ADVANCED (INTERVIEW KILLER)

### 11. Custom Sorting with Complex Conditions
Sort employees:
1. First by salary DESC
2. Then by name ASC

```java
List<Employee> sorted =
    employees.stream()
        .sorted(Comparator
            .comparing(Employee::getSalary).reversed()
            .thenComparing(Employee::getName))
        .collect(Collectors.toList());
```

---

### 12. Parallel Stream Performance Trap
```java
list.parallelStream()
    .forEach(System.out::println);
```
**Problem:**
* Order is not guaranteed
* Debugging becomes hard

**Fix:**
```java
list.parallelStream()
    .forEachOrdered(System.out::println);
```