# Module 03 – Control Flow

## Table of Contents
1. [if / else if / else](#1-if--else-if--else)
2. [Switch Statement](#2-switch-statement)
3. [Switch Expressions (Java 14+)](#3-switch-expressions-java-14)
4. [for Loop](#4-for-loop)
5. [Enhanced for Loop (for-each)](#5-enhanced-for-loop-for-each)
6. [while Loop](#6-while-loop)
7. [do-while Loop](#7-do-while-loop)
8. [break, continue, and Labels](#8-break-continue-and-labels)
9. [Pattern Matching in switch (Java 21+)](#9-pattern-matching-in-switch-java-21)
10. [Exercises](#10-exercises)

---

## 1. if / else if / else

```java
int score = 85;

if (score >= 90) {
    System.out.println("Grade: A");
} else if (score >= 80) {
    System.out.println("Grade: B");
} else if (score >= 70) {
    System.out.println("Grade: C");
} else {
    System.out.println("Grade: F");
}
// Output: Grade: B
```

### Nested if
```java
int age = 20;
boolean hasLicense = true;

if (age >= 18) {
    if (hasLicense) {
        System.out.println("Can drive");
    } else {
        System.out.println("Too young and no license");
    }
}
```

### One-liner (no braces) – use with caution
```java
if (age >= 18) System.out.println("Adult");
// Avoid this style for multi-statement logic
```

---

## 2. Switch Statement

```java
int day = 3;

switch (day) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    case 3:
        System.out.println("Wednesday");
        break;
    case 6:
    case 7:
        System.out.println("Weekend");
        break;
    default:
        System.out.println("Other day");
}
// Output: Wednesday
```

> **Fall-through**: Without `break`, execution continues into the next case. This is often a bug, but can be intentional (like grouping 6 and 7 above).

Switch can operate on: `byte`, `short`, `int`, `char`, `String`, `enum`.

---

## 3. Switch Expressions (Java 14+)

Switch **expressions** return a value and use `->` (arrow syntax) to eliminate fall-through:

```java
int day = 3;

String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4 -> "Thursday";
    case 5 -> "Friday";
    case 6, 7 -> "Weekend";
    default -> "Unknown";
};

System.out.println(dayName); // Wednesday
```

### Multi-statement blocks with `yield`
```java
int day = 6;
String type = switch (day) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7 -> {
        System.out.println("It's the weekend!");
        yield "Weekend";   // yield returns the value from the block
    }
    default -> throw new IllegalArgumentException("Invalid day: " + day);
};
```

### Switch with String
```java
String command = "start";

switch (command) {
    case "start" -> System.out.println("Starting...");
    case "stop"  -> System.out.println("Stopping...");
    case "pause" -> System.out.println("Paused.");
    default      -> System.out.println("Unknown command: " + command);
}
```

---

## 4. for Loop

```java
// Basic for loop
for (int i = 0; i < 5; i++) {
    System.out.print(i + " ");
}
// Output: 0 1 2 3 4

// Counting down
for (int i = 10; i >= 0; i -= 2) {
    System.out.print(i + " ");
}
// Output: 10 8 6 4 2 0

// Iterating over an array
int[] numbers = {10, 20, 30, 40, 50};
for (int i = 0; i < numbers.length; i++) {
    System.out.println("Index " + i + ": " + numbers[i]);
}

// Infinite loop (use with care)
// for (;;) { ... }
```

---

## 5. Enhanced for Loop (for-each)

Simpler syntax for iterating over arrays and collections:

```java
int[] numbers = {1, 2, 3, 4, 5};

for (int num : numbers) {
    System.out.print(num + " ");
}
// Output: 1 2 3 4 5

// With a List
import java.util.List;
List<String> fruits = List.of("Apple", "Banana", "Cherry");

for (String fruit : fruits) {
    System.out.println(fruit);
}
```

> **Limitation**: You don't have access to the index in a for-each loop. Use the classic `for` loop if you need the index.

---

## 6. while Loop

```java
// Execute while condition is true
int count = 0;
while (count < 5) {
    System.out.print(count + " ");
    count++;
}
// Output: 0 1 2 3 4

// Reading until a sentinel value
import java.util.Scanner;
Scanner scanner = new Scanner(System.in);
int sum = 0;
int input = scanner.nextInt();

while (input != -1) {
    sum += input;
    input = scanner.nextInt();
}
System.out.println("Sum: " + sum);
```

---

## 7. do-while Loop

Executes the body **at least once**, then checks the condition:

```java
int n = 10;
do {
    System.out.print(n + " ");
    n--;
} while (n > 0);
// Output: 10 9 8 7 6 5 4 3 2 1

// Useful for menu-driven programs
int choice;
do {
    System.out.println("1. Start");
    System.out.println("2. Stop");
    System.out.println("0. Exit");
    choice = new Scanner(System.in).nextInt();
} while (choice != 0);
```

---

## 8. break, continue, and Labels

### `break` – exit the loop
```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break;
    System.out.print(i + " ");
}
// Output: 0 1 2 3 4
```

### `continue` – skip current iteration
```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;
    System.out.print(i + " ");
}
// Output: 1 3 5 7 9
```

### Labels – break/continue outer loops
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer;  // skip to next iteration of outer loop
        System.out.println(i + "," + j);
    }
}
// Output:
// 0,0
// 1,0
// 2,0
```

---

## 9. Pattern Matching in switch (Java 21+)

Java 21 introduced **pattern matching for switch** (finalized in Java 21, available in Java 23):

```java
Object obj = "Hello, Java 23!";

String result = switch (obj) {
    case Integer i -> "Integer: " + i;
    case Double  d -> "Double: "  + d;
    case String  s -> "String: "  + s;
    case null      -> "null value";
    default        -> "Other type: " + obj.getClass().getSimpleName();
};

System.out.println(result); // String: Hello, Java 23!
```

### Guarded patterns with `when`
```java
Object value = 42;

String description = switch (value) {
    case Integer i when i > 0  -> "Positive integer: " + i;
    case Integer i when i < 0  -> "Negative integer: " + i;
    case Integer i             -> "Zero";
    case String  s when s.isEmpty() -> "Empty string";
    case String  s             -> "String: " + s;
    default                    -> "Something else";
};

System.out.println(description); // Positive integer: 42
```

### Deconstruction patterns with records
```java
record Point(int x, int y) {}

Object shape = new Point(3, 4);

String desc = switch (shape) {
    case Point(int x, int y) when x == 0 && y == 0 -> "Origin";
    case Point(int x, int y) -> "Point at (" + x + ", " + y + ")";
    default -> "Not a point";
};

System.out.println(desc); // Point at (3, 4)
```

---

## 10. Exercises

### Exercise 1 – Grade Classifier
Write a program that takes a score (0–100) and prints the corresponding letter grade:
- 90–100 → A
- 80–89  → B
- 70–79  → C
- 60–69  → D
- Below 60 → F

### Exercise 2 – FizzBuzz
Print numbers from 1 to 100. For multiples of 3 print "Fizz", for multiples of 5 print "Buzz", for multiples of both print "FizzBuzz".

### Exercise 3 – Multiplication Table
Using nested `for` loops, print the multiplication table (1–10):
```
1  2  3  4  5  6  7  8  9  10
2  4  6  8  10 12 14 16 18 20
...
```

### Exercise 4 – Palindrome Check
Write a loop that checks whether a given string is a palindrome (reads the same forwards and backwards). For example: "racecar", "level".

### Exercise 5 – Switch Expression
Rewrite this classic `switch` statement as a modern switch expression:
```java
int month = 4;
int days;
switch (month) {
    case 2: days = 28; break;
    case 4: case 6: case 9: case 11: days = 30; break;
    default: days = 31;
}
```

### Exercise 6 – Pattern Matching
Given the following array:
```java
Object[] items = {42, "hello", 3.14, null, true, -7};
```
Use a pattern-matching switch to categorize each element as:
- Positive number, Negative number, Zero, Non-empty string, Empty/null, Boolean, or Other.

### Exercise 7 – Fibonacci Sequence
Using a `while` loop, print the first 20 numbers of the Fibonacci sequence:
```
0, 1, 1, 2, 3, 5, 8, 13, ...
```

---

> ⬅️ Previous: [Module 02 – Variables, Data Types & Operators](02-variables-datatypes-operators.md) | ➡️ Next: [Module 04 – OOP: Classes & Objects](04-oop-classes-and-objects.md)
