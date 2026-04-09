# Module 02 – Variables, Data Types & Operators

## Table of Contents
1. [Variables and Declaration](#1-variables-and-declaration)
2. [Primitive Data Types](#2-primitive-data-types)
3. [Reference Types](#3-reference-types)
4. [Type Inference with `var`](#4-type-inference-with-var)
5. [Literals](#5-literals)
6. [Type Casting](#6-type-casting)
7. [Operators](#7-operators)
8. [String Basics](#8-string-basics)
9. [Exercises](#9-exercises)

---

## 1. Variables and Declaration

A **variable** is a named container for storing data.

```java
// Syntax: <type> <name> = <value>;
int age = 25;
String name = "Alice";
double salary = 5_500.75;    // underscores improve readability (Java 7+)
boolean isActive = true;
```

### Variable naming rules
- Must start with a letter, `$`, or `_`
- Cannot be a Java keyword (e.g., `int`, `class`)
- Case-sensitive: `myVar` ≠ `myvar`
- Convention: **camelCase** for variables/methods, **PascalCase** for classes, **UPPER_SNAKE_CASE** for constants

### `final` – constants
```java
final double PI = 3.14159265358979;
// PI = 3.0;  // Compile error – cannot reassign a final variable
```

---

## 2. Primitive Data Types

Java has **8 primitive types** (stored on the stack, not on the heap):

| Type | Size | Default | Range / Notes |
|------|------|---------|---------------|
| `byte` | 1 byte | 0 | -128 to 127 |
| `short` | 2 bytes | 0 | -32,768 to 32,767 |
| `int` | 4 bytes | 0 | ~-2.1 billion to ~2.1 billion |
| `long` | 8 bytes | 0L | ~-9.2 × 10^18 to ~9.2 × 10^18 |
| `float` | 4 bytes | 0.0f | ~7 decimal digits precision |
| `double` | 8 bytes | 0.0d | ~15 decimal digits precision |
| `char` | 2 bytes | '\u0000' | Unicode character (0–65,535) |
| `boolean` | 1 bit* | false | `true` or `false` |

> *JVM-dependent; typically stored as a full int in practice.

```java
byte b = 100;
short s = 30_000;
int i = 1_000_000;
long l = 10_000_000_000L;      // suffix 'L' required for long literals

float f = 3.14f;               // suffix 'f' required for float literals
double d = 3.141592653589793;

char c = 'A';                  // single quotes for char
char unicode = '\u0041';       // also 'A'

boolean flag = true;
```

---

## 3. Reference Types

Reference types store a **reference (address) to an object** in the heap.

```java
String greeting = "Hello";        // String object on the heap
int[] numbers = {1, 2, 3};        // Array object
StringBuilder sb = new StringBuilder();  // custom object
```

Key distinction:

```java
// Primitive – copy of value
int a = 5;
int b = a;
b = 10;
System.out.println(a); // 5  (unchanged)

// Reference – copy of address
int[] arr1 = {1, 2, 3};
int[] arr2 = arr1;       // both point to same array
arr2[0] = 99;
System.out.println(arr1[0]); // 99  (changed!)
```

### `null`
A reference variable that points to nothing:

```java
String s = null;
System.out.println(s);       // prints: null
System.out.println(s.length()); // NullPointerException!
```

---

## 4. Type Inference with `var`

Since Java 10, **`var`** lets the compiler infer the type from the initializer:

```java
var message = "Hello, Java 23!";  // inferred as String
var count   = 42;                  // inferred as int
var prices  = new ArrayList<Double>(); // inferred as ArrayList<Double>

// var must be initialized
// var x;  // Compile error

// var cannot be used for class fields, method params, or return types
```

`var` does **not** make Java dynamically typed – the type is fixed at compile time.

---

## 5. Literals

```java
// Integer literals
int decimal = 255;
int hex      = 0xFF;       // hexadecimal
int octal    = 0377;       // octal
int binary   = 0b11111111; // binary (Java 7+)
int readable = 1_000_000;  // underscores (Java 7+)

// Floating-point literals
double d = 1.23e4;  // scientific notation = 12300.0
float  f = 1.23e4f;

// Character literals
char newline = '\n';
char tab     = '\t';
char quote   = '\'';

// String literals
String s = "Hello\nWorld";    // escape sequences in strings
String raw = """
        Hello
        World
        """;                   // text block (Java 15+)
```

---

## 6. Type Casting

### Widening (automatic – no data loss)
```java
int i = 100;
long l = i;        // int → long (automatic)
double d = i;      // int → double (automatic)
```

### Narrowing (explicit – possible data loss)
```java
double d = 9.99;
int i = (int) d;   // explicit cast – result is 9 (truncated, not rounded)

long l = 300L;
byte b = (byte) l; // 300 % 256 = 44
```

### Wrapper type conversions
```java
// Boxing: primitive → wrapper object
Integer wrapped = Integer.valueOf(42);
Integer autoboxed = 42;               // autoboxing (Java 5+)

// Unboxing: wrapper object → primitive
int primitive = wrapped;              // auto-unboxing

// Parsing from String
int parsed = Integer.parseInt("123");
double parsedD = Double.parseDouble("3.14");

// Converting to String
String s = String.valueOf(42);
String s2 = Integer.toString(42);
String s3 = 42 + "";   // quick but less efficient
```

---

## 7. Operators

### Arithmetic
```java
int a = 10, b = 3;
System.out.println(a + b);   // 13  addition
System.out.println(a - b);   // 7   subtraction
System.out.println(a * b);   // 30  multiplication
System.out.println(a / b);   // 3   integer division (truncates)
System.out.println(a % b);   // 1   modulo (remainder)

double result = 10.0 / 3;    // 3.3333... (at least one operand double)
```

### Assignment & Compound Assignment
```java
int x = 5;
x += 3;   // x = x + 3  → 8
x -= 2;   // x = x - 2  → 6
x *= 4;   // x = x * 4  → 24
x /= 6;   // x = x / 6  → 4
x %= 3;   // x = x % 3  → 1
```

### Increment & Decrement
```java
int n = 5;
System.out.println(n++);  // 5  (uses then increments)
System.out.println(n);    // 6
System.out.println(++n);  // 7  (increments then uses)
System.out.println(n--);  // 7  (uses then decrements)
System.out.println(n);    // 6
```

### Comparison (relational)
```java
int a = 5, b = 10;
System.out.println(a == b);  // false
System.out.println(a != b);  // true
System.out.println(a <  b);  // true
System.out.println(a <= b);  // true
System.out.println(a >  b);  // false
System.out.println(a >= b);  // false
```

### Logical
```java
boolean p = true, q = false;
System.out.println(p && q);  // false  (AND – both must be true)
System.out.println(p || q);  // true   (OR – at least one must be true)
System.out.println(!p);      // false  (NOT)

// Short-circuit evaluation
int x = 0;
boolean result = (x != 0) && (10 / x > 1); // safe – right side not evaluated
```

### Bitwise
```java
int a = 0b1010;  // 10
int b = 0b1100;  // 12
System.out.println(a & b);   // 0b1000 = 8   (AND)
System.out.println(a | b);   // 0b1110 = 14  (OR)
System.out.println(a ^ b);   // 0b0110 = 6   (XOR)
System.out.println(~a);      // -11 (complement)
System.out.println(a << 1);  // 20  (left shift)
System.out.println(a >> 1);  // 5   (right shift)
System.out.println(a >>> 1); // 5   (unsigned right shift)
```

### Ternary
```java
int score = 72;
String grade = (score >= 60) ? "Pass" : "Fail";
System.out.println(grade); // Pass
```

### Operator Precedence (high to low, simplified)
```
postfix:       expr++ expr--
unary:         ++expr --expr +expr -expr ~ !
multiplicative: * / %
additive:       + -
shift:          << >> >>>
relational:     < > <= >= instanceof
equality:       == !=
bitwise AND:    &
bitwise XOR:    ^
bitwise OR:     |
logical AND:    &&
logical OR:     ||
ternary:        ? :
assignment:     = += -= *= /= %= &= ^= |= <<= >>= >>>=
```

---

## 8. String Basics

```java
String s = "Hello, Java!";

// Length
System.out.println(s.length());     // 12

// Character at index
System.out.println(s.charAt(0));    // H

// Substring
System.out.println(s.substring(7)); // Java!
System.out.println(s.substring(7, 11)); // Java

// Search
System.out.println(s.indexOf("Java")); // 7
System.out.println(s.contains("Java")); // true

// Case
System.out.println(s.toUpperCase()); // HELLO, JAVA!
System.out.println(s.toLowerCase()); // hello, java!

// Trimming
String padded = "  hello  ";
System.out.println(padded.trim());   // "hello"
System.out.println(padded.strip()); // "hello" (Unicode-aware, Java 11+)

// Replace
System.out.println(s.replace("Java", "World")); // Hello, World!

// Split
String csv = "a,b,c";
String[] parts = csv.split(",");   // ["a", "b", "c"]

// String comparison
String a = "hello";
String b = "hello";
System.out.println(a == b);        // true (string pool, same literal)
System.out.println(a.equals(b));   // true (always use .equals() for safety)

String c = new String("hello");
System.out.println(a == c);        // false (different objects)
System.out.println(a.equals(c));   // true

// String concatenation and formatting
String name = "Alice";
int age = 30;
String msg1 = "Name: " + name + ", Age: " + age;
String msg2 = String.format("Name: %s, Age: %d", name, age);
String msg3 = "Name: %s, Age: %d".formatted(name, age); // Java 15+
System.out.println(msg3); // Name: Alice, Age: 30

// Text blocks (Java 15+)
String json = """
        {
            "name": "Alice",
            "age": 30
        }
        """;
```

---

## 9. Exercises

### Exercise 1 – Data Types
Declare one variable of each primitive type. Print all of them.

### Exercise 2 – Temperature Converter
Write a program that:
- Declares a `double` variable `celsius = 100.0`
- Converts it to Fahrenheit using the formula: `F = (C * 9/5) + 32`
- Prints both values formatted to 2 decimal places

Expected output:
```
100.00°C = 212.00°F
```

### Exercise 3 – Integer Overflow
What happens when you do:
```java
byte b = 127;
b++;
System.out.println(b);
```
Run it and explain the result.

### Exercise 4 – String Operations
Given:
```java
String sentence = "  Learning Java 23 is fun!  ";
```
Print:
1. The sentence without leading/trailing spaces
2. The sentence in uppercase
3. The number of characters (after trimming)
4. Whether the trimmed sentence starts with "Learning"
5. The sentence with "fun" replaced by "awesome"

### Exercise 5 – Operators Challenge
Without running the code, predict the output of:
```java
int x = 10;
int y = x++ + ++x;
System.out.println("x=" + x + ", y=" + y);
```
Then verify by running it.

### Exercise 6 – `var` Usage
Rewrite the following using `var`:
```java
ArrayList<String> names = new ArrayList<String>();
String greeting = "Hello";
int count = names.size();
```

---

> ⬅️ Previous: [Module 01 – Introduction](01-introduction-and-setup.md) | ➡️ Next: [Module 03 – Control Flow](03-control-flow.md)
