# Module 10 – Functional Programming: Lambdas & Streams

## Table of Contents
1. [Lambda Expressions](#1-lambda-expressions)
2. [Method References](#2-method-references)
3. [Functional Interfaces Deep Dive](#3-functional-interfaces-deep-dive)
4. [Stream API](#4-stream-api)
5. [Collectors](#5-collectors)
6. [Optional](#6-optional)
7. [Function Composition](#7-function-composition)
8. [Exercises](#8-exercises)

---

## 1. Lambda Expressions

A **lambda expression** is a concise way to represent an anonymous function (a block of code to pass around):

```java
// Syntax: (parameters) -> expression
//         (parameters) -> { statements; }

// No parameters
Runnable greet = () -> System.out.println("Hello!");
greet.run();  // Hello!

// One parameter (parentheses optional)
Consumer<String> print = s -> System.out.println(s);
print.accept("Lambda!"); // Lambda!

// Multiple parameters
Comparator<String> byLength = (a, b) -> a.length() - b.length();

// Block body (multiple statements)
Function<Integer, String> classify = n -> {
    if      (n < 0) return "negative";
    else if (n == 0) return "zero";
    else             return "positive";
};

System.out.println(classify.apply(-5));  // negative
System.out.println(classify.apply(0));   // zero
System.out.println(classify.apply(7));   // positive
```

### Variable capture
Lambdas can capture variables from the enclosing scope, but they must be **effectively final**:

```java
String prefix = "Hello, ";  // effectively final

Consumer<String> greeter = name -> System.out.println(prefix + name);
greeter.accept("Alice");  // Hello, Alice

// prefix = "Hi, ";  // Would break compilation – prefix must be effectively final
```

---

## 2. Method References

Method references are a shorthand for lambdas that simply call an existing method:

| Type | Syntax | Lambda Equivalent |
|------|--------|------------------|
| Static method | `ClassName::staticMethod` | `x -> ClassName.staticMethod(x)` |
| Instance method (specific instance) | `instance::method` | `x -> instance.method(x)` |
| Instance method (arbitrary instance) | `ClassName::instanceMethod` | `(x) -> x.instanceMethod()` |
| Constructor | `ClassName::new` | `x -> new ClassName(x)` |

```java
import java.util.*;
import java.util.function.*;

// Static method reference
Function<String, Integer> parse = Integer::parseInt;
System.out.println(parse.apply("42")); // 42

// Instance method on a specific object
String prefix = "Hello, ";
Function<String, String> greet = prefix::concat;
System.out.println(greet.apply("World")); // Hello, World

// Instance method on arbitrary instance
Function<String, String> upper = String::toUpperCase;
System.out.println(upper.apply("java")); // JAVA

Predicate<String> isEmpty = String::isEmpty;
System.out.println(isEmpty.test(""));   // true
System.out.println(isEmpty.test("hi")); // false

// Constructor reference
Supplier<ArrayList<String>> listFactory = ArrayList::new;
ArrayList<String> list = listFactory.get();

Function<String, StringBuilder> sbFactory = StringBuilder::new;
StringBuilder sb = sbFactory.apply("Hello");

// Used in streams
List<String> names = List.of("alice", "bob", "carol");
names.stream()
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

---

## 3. Functional Interfaces Deep Dive

```java
import java.util.function.*;

// Supplier – produces a value
Supplier<Double> random = Math::random;
Supplier<List<String>> emptyList = ArrayList::new;

// Consumer
Consumer<String> printer  = System.out::println;
Consumer<String> logger   = msg -> System.err.println("[LOG] " + msg);
Consumer<String> both     = printer.andThen(logger);  // chaining

// BiConsumer
BiConsumer<String, Integer> printEntry = (k, v) ->
    System.out.println(k + " = " + v);

// Function
Function<String, Integer> length   = String::length;
Function<Integer, Boolean> isEven  = n -> n % 2 == 0;
Function<String, Boolean> evenLength = length.andThen(isEven); // compose

System.out.println(evenLength.apply("Java"));  // true (4 chars)

// BiFunction
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<String, String, String> concat = String::concat;

// Predicate
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> shortStr = s -> s.length() < 5;
Predicate<String> valid    = notEmpty.and(shortStr);
Predicate<String> invalid  = valid.negate();

System.out.println(valid.test("Hi"));         // true
System.out.println(valid.test(""));           // false
System.out.println(valid.test("Hello World")); // false (too long)

// UnaryOperator (Function<T,T>)
UnaryOperator<String> trim      = String::trim;
UnaryOperator<String> toUpper   = String::toUpperCase;
UnaryOperator<String> normalize = trim.andThen(toUpper);

System.out.println(normalize.apply("  hello  ")); // HELLO

// BinaryOperator (BiFunction<T,T,T>)
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
System.out.println(max.apply(3, 7)); // 7
```

---

## 4. Stream API

A **Stream** is a sequence of elements supporting sequential and parallel aggregate operations. Streams are **lazy** – intermediate operations are only evaluated when a terminal operation is called.

```
Source → intermediate ops (lazy) → terminal op (triggers processing)
```

### Creating streams
```java
import java.util.stream.*;

// From collection
Stream<String> fromList = List.of("a", "b", "c").stream();

// From array
Stream<String> fromArray = Arrays.stream(new String[]{"x", "y"});

// From values
Stream<Integer> of = Stream.of(1, 2, 3, 4, 5);

// Infinite streams
Stream<Integer> iterate  = Stream.iterate(0, n -> n + 2);  // 0, 2, 4, 6, ...
Stream<Double>  generate = Stream.generate(Math::random);

// Range streams (primitive)
IntStream range = IntStream.range(1, 6);       // 1, 2, 3, 4, 5
IntStream rangeClosed = IntStream.rangeClosed(1, 5); // same
LongStream longs = LongStream.of(100L, 200L, 300L);
```

### Intermediate operations (lazy – return a Stream)
```java
List<String> names = List.of("Alice", "Bob", "Carol", "Dave", "Eve", "Frank");

names.stream()
    .filter(s -> s.length() > 3)          // filter by condition
    .map(String::toUpperCase)              // transform elements
    .sorted()                              // natural sort
    .distinct()                            // remove duplicates
    .limit(3)                              // take at most 3
    .skip(1)                               // skip first 1
    .peek(s -> System.out.println("peek: " + s)) // debug intermediate state
    .forEach(System.out::println);         // terminal: consume

// flatMap – flatten nested streams
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4), List.of(5));
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
System.out.println(flat); // [1, 2, 3, 4, 5]
```

### Terminal operations (trigger execution)
```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// forEach
numbers.stream().forEach(System.out::print);

// collect – gather into a collection
List<Integer> evens  = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // [2, 4, 6, 8, 10]

// count
long count = numbers.stream().filter(n -> n > 5).count(); // 5

// reduce
int sum = numbers.stream().reduce(0, Integer::sum);                  // 55
Optional<Integer> product = numbers.stream().reduce((a, b) -> a * b); // 3628800

// min / max
Optional<Integer> min = numbers.stream().min(Integer::compare); // Optional[1]
Optional<Integer> max = numbers.stream().max(Integer::compare); // Optional[10]

// findFirst / findAny
Optional<Integer> first = numbers.stream().filter(n -> n > 5).findFirst(); // 6

// anyMatch / allMatch / noneMatch
boolean anyEven  = numbers.stream().anyMatch(n -> n % 2 == 0);  // true
boolean allPositive = numbers.stream().allMatch(n -> n > 0);     // true
boolean noneNeg  = numbers.stream().noneMatch(n -> n < 0);       // true

// toArray
Integer[] arr = numbers.stream().toArray(Integer[]::new);

// Numeric statistics
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();
System.out.println("Sum: " + stats.getSum());     // 55
System.out.println("Avg: " + stats.getAverage()); // 5.5
System.out.println("Max: " + stats.getMax());     // 10
```

---

## 5. Collectors

```java
import java.util.stream.Collectors;
import java.util.*;

record Person(String name, int age, String city) {}

List<Person> people = List.of(
    new Person("Alice", 30, "São Paulo"),
    new Person("Bob",   25, "Rio de Janeiro"),
    new Person("Carol", 35, "São Paulo"),
    new Person("Dave",  28, "Curitiba"),
    new Person("Eve",   22, "São Paulo")
);

// Collect to list
List<String> names = people.stream()
    .map(Person::name)
    .collect(Collectors.toList());

// Collect to set
Set<String> cities = people.stream()
    .map(Person::city)
    .collect(Collectors.toSet());

// Join strings
String joined = people.stream()
    .map(Person::name)
    .collect(Collectors.joining(", ", "[", "]"));
System.out.println(joined); // [Alice, Bob, Carol, Dave, Eve]

// Grouping
Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::city));
byCity.forEach((city, persons) ->
    System.out.println(city + ": " + persons.stream().map(Person::name).toList()));

// Count by city
Map<String, Long> countByCity = people.stream()
    .collect(Collectors.groupingBy(Person::city, Collectors.counting()));
System.out.println(countByCity); // {São Paulo=3, Rio de Janeiro=1, Curitiba=1}

// Average age by city
Map<String, Double> avgAgeByCity = people.stream()
    .collect(Collectors.groupingBy(Person::city,
             Collectors.averagingInt(Person::age)));

// Partitioning (split into two groups: true/false)
Map<Boolean, List<Person>> over30 = people.stream()
    .collect(Collectors.partitioningBy(p -> p.age() >= 30));
System.out.println("Over 30: " + over30.get(true).stream().map(Person::name).toList());

// toMap
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(Person::name, Person::age));

// Summarizing statistics
IntSummaryStatistics ageSummary = people.stream()
    .collect(Collectors.summarizingInt(Person::age));
System.out.println("Average age: " + ageSummary.getAverage());

// Unmodifiable collectors (Java 10+)
List<String> unmodifiable = people.stream()
    .map(Person::name)
    .collect(Collectors.toUnmodifiableList());
```

---

## 6. Optional

`Optional<T>` is a container that may or may not contain a non-null value. It avoids `NullPointerException` by making optionality explicit:

```java
import java.util.Optional;

// Creating
Optional<String> empty    = Optional.empty();
Optional<String> present  = Optional.of("Hello");        // throws NPE if null
Optional<String> nullable = Optional.ofNullable(null);   // safe with null

// Checking
System.out.println(present.isPresent());   // true
System.out.println(empty.isEmpty());       // true

// Getting the value
String value = present.get();               // throws if empty – avoid using directly

// Safe patterns
present.ifPresent(s -> System.out.println("Value: " + s));

String result = empty.orElse("default");                   // "default"
String result2 = empty.orElseGet(() -> computeDefault());  // lazy
String result3 = empty.orElseThrow(() -> new RuntimeException("Not found"));

// Transforming
Optional<Integer> length = present.map(String::length);
System.out.println(length); // Optional[5]

Optional<String> upper = present.map(String::toUpperCase);
System.out.println(upper.orElse("")); // HELLO

// Filtering
Optional<String> longStr = present.filter(s -> s.length() > 10);
System.out.println(longStr); // Optional.empty

// FlatMap – avoid Optional<Optional<T>>
Optional<String> city = Optional.of("user123")
    .flatMap(id -> findUser(id))          // returns Optional<User>
    .flatMap(user -> user.getCity());     // returns Optional<String>

// or() – alternative Optional (Java 9+)
Optional<String> fallback = empty.or(() -> Optional.of("fallback"));
System.out.println(fallback); // Optional[fallback]

// ifPresentOrElse (Java 9+)
present.ifPresentOrElse(
    s -> System.out.println("Found: " + s),
    () -> System.out.println("Not found")
);
```

---

## 7. Function Composition

```java
import java.util.function.*;

Function<String, String> trim    = String::trim;
Function<String, String> lower   = String::toLowerCase;
Function<String, Boolean> valid  = s -> !s.isEmpty();

// compose – apply other first, then this
Function<String, String> trimThenLower = lower.compose(trim); // trim first, then lower

// andThen – apply this first, then other
Function<String, String> lowerThenTrim = lower.andThen(trim); // lower first, then trim

System.out.println(trimThenLower.apply("  HELLO  ")); // "hello"
System.out.println(lowerThenTrim.apply("  HELLO  ")); // "hello" (same here)

// Building a pipeline
Function<String, String> sanitize = trim
    .andThen(lower)
    .andThen(s -> s.replaceAll("[^a-z0-9]", ""));

System.out.println(sanitize.apply("  Hello, World! 123  ")); // helloworld123

// Predicate composition
Predicate<String> notEmpty  = s -> !s.isEmpty();
Predicate<String> noSpaces  = s -> !s.contains(" ");
Predicate<String> shortEnough = s -> s.length() <= 20;

Predicate<String> validUsername = notEmpty.and(noSpaces).and(shortEnough);

System.out.println(validUsername.test("alice"));          // true
System.out.println(validUsername.test("alice smith"));    // false (has space)
System.out.println(validUsername.test(""));               // false (empty)
```

---

## 8. Exercises

### Exercise 1 – Stream Pipeline
Given a list of words, write a single stream pipeline that:
1. Filters out words with fewer than 4 characters
2. Converts each word to title case (first letter uppercase, rest lowercase)
3. Removes duplicates
4. Sorts alphabetically
5. Collects to a `List<String>`

### Exercise 2 – Data Analysis
Given a list of sales records `record Sale(String product, String region, double amount)`:
1. Total sales amount
2. Sales by region (Map<String, Double>)
3. Top 3 products by total sales
4. Number of sales above the average

### Exercise 3 – Optional Chain
Write a method that:
```java
Optional<String> getUserEmailDomain(int userId)
```
Which uses a chain of `flatMap`/`map` calls through `findUserById`, `getEmail`, and splitting the domain from the email.

### Exercise 4 – Custom Collector
Implement a custom `Collector<String, ?, Map<Integer, List<String>>>` that groups strings by their length.

### Exercise 5 – Parallel Streams
Take a list of 10,000 random integers and use a **parallel stream** to:
1. Find all prime numbers
2. Compare execution time with a sequential stream

Use `System.nanoTime()` for timing.

### Exercise 6 – FlatMap Magic
Given a list of sentences, flatten all words into one list, remove stopwords ("the", "a", "an", "is", "are"), and return the top 5 most frequent words with their counts.

---

> ⬅️ Previous: [Module 09 – Exception Handling](09-exception-handling.md) | ➡️ Next: [Module 11 – Java I/O & NIO](11-io-and-nio.md)
