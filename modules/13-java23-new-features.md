# Module 13 – Java 23 New Features

## Table of Contents
1. [Overview of Java 23 (September 2024)](#1-overview-of-java-23-september-2024)
2. [Records (Java 16 – Stable)](#2-records-java-16--stable)
3. [Sealed Classes & Interfaces (Java 17 – Stable)](#3-sealed-classes--interfaces-java-17--stable)
4. [Pattern Matching for switch (Java 21 – Stable)](#4-pattern-matching-for-switch-java-21--stable)
5. [Record Patterns (Java 21 – Stable)](#5-record-patterns-java-21--stable)
6. [Text Blocks (Java 15 – Stable)](#6-text-blocks-java-15--stable)
7. [Unnamed Classes & Instance Main (Java 23 – Preview)](#7-unnamed-classes--instance-main-java-23--preview)
8. [Primitive Types in Patterns (Java 23 – Preview)](#8-primitive-types-in-patterns-java-23--preview)
9. [String Templates (withdrawn – alternative approaches)](#9-string-templates-withdrawn--alternative-approaches)
10. [Other Notable Changes](#10-other-notable-changes)
11. [Exercises](#11-exercises)

---

## 1. Overview of Java 23 (September 2024)

Java 23 is a **Standard-Term Support (STS)** release (not LTS). It contains:

| JEP | Feature | Status |
|-----|---------|--------|
| JEP 455 | Primitive Types in Patterns | Preview |
| JEP 461 | Stream Gatherers | Preview |
| JEP 462 | Structured Concurrency | Second Preview |
| JEP 463 | Implicitly Declared Classes and Instance Main Methods | Second Preview |
| JEP 464 | Scoped Values | Second Preview |
| JEP 465 | Vector API | Eighth Incubator |
| JEP 466 | Class-File API | Second Preview |
| JEP 467 | Markdown Documentation Comments | Final |
| JEP 469 | ZGC: Generational Mode by Default | Final |
| JEP 471 | Deprecate Memory-Access Methods in `sun.misc.Unsafe` | Final |
| JEP 473 | Stream Gatherers | Second Preview |
| JEP 474 | ZGC: Generational Mode by Default | Final |

Features from earlier Java versions that are now **standard** and available by default:
- Records (Java 16), Sealed Classes (Java 17), Pattern Matching (Java 21), Virtual Threads (Java 21)

---

## 2. Records (Java 16 – Stable)

Already covered in Module 04, but here's a comprehensive recap with advanced features:

```java
// Basic record
public record Point(double x, double y) {

    // Compact constructor – for validation
    public Point {
        if (Double.isNaN(x) || Double.isNaN(y)) {
            throw new IllegalArgumentException("Coordinates cannot be NaN");
        }
    }

    // Custom method
    public double distanceTo(Point other) {
        return Math.sqrt(Math.pow(x - other.x, 2) + Math.pow(y - other.y, 2));
    }

    // Static factory
    public static Point origin() { return new Point(0, 0); }

    // "Wither" pattern – record with one field changed
    public Point withX(double newX) { return new Point(newX, y); }
    public Point withY(double newY) { return new Point(x, newY); }
}
```

```java
Point p1 = new Point(3, 4);
Point p2 = Point.origin();

System.out.println(p1);               // Point[x=3.0, y=4.0]
System.out.println(p1.distanceTo(p2)); // 5.0

// Records are immutable value objects
Point p3 = p1.withX(6);
System.out.println(p3);               // Point[x=6.0, y=4.0]
System.out.println(p1);               // Point[x=3.0, y=4.0] – unchanged

// Records in collections
var points = List.of(new Point(1, 2), new Point(3, 4), new Point(0, 0));
points.stream().sorted(Comparator.comparingDouble(p -> p.distanceTo(Point.origin())))
      .forEach(System.out::println);
```

### Nested records
```java
public record Address(String street, String city, String country) {}

public record Person(String name, int age, Address address) {}
```

---

## 3. Sealed Classes & Interfaces (Java 17 – Stable)

Already introduced in Module 06. Here's a more complete example:

```java
// Sealed class hierarchy for a simple expression language
public sealed interface Expr
    permits Expr.Num, Expr.Add, Expr.Mul, Expr.Neg {

    record Num(double value)          implements Expr {}
    record Add(Expr left, Expr right) implements Expr {}
    record Mul(Expr left, Expr right) implements Expr {}
    record Neg(Expr expr)             implements Expr {}
}
```

```java
// Evaluate using exhaustive switch (no default needed!)
public static double eval(Expr expr) {
    return switch (expr) {
        case Expr.Num n      -> n.value();
        case Expr.Add a      -> eval(a.left()) + eval(a.right());
        case Expr.Mul m      -> eval(m.left()) * eval(m.right());
        case Expr.Neg n      -> -eval(n.expr());
    };
}

// Pretty print
public static String prettyPrint(Expr expr) {
    return switch (expr) {
        case Expr.Num n      -> String.valueOf(n.value());
        case Expr.Add a      -> "(" + prettyPrint(a.left()) + " + " + prettyPrint(a.right()) + ")";
        case Expr.Mul m      -> "(" + prettyPrint(m.left()) + " * " + prettyPrint(m.right()) + ")";
        case Expr.Neg n      -> "-" + prettyPrint(n.expr());
    };
}
```

```java
// (2 + 3) * -(4)
Expr expression = new Expr.Mul(
    new Expr.Add(new Expr.Num(2), new Expr.Num(3)),
    new Expr.Neg(new Expr.Num(4))
);

System.out.println(prettyPrint(expression)); // ((2.0 + 3.0) * -4.0)
System.out.println(eval(expression));        // -20.0
```

---

## 4. Pattern Matching for switch (Java 21 – Stable)

Full pattern matching for switch was finalized in Java 21 and available in Java 23:

```java
// Type patterns
Object obj = getSomeValue();

String formatted = switch (obj) {
    case null           -> "null";
    case Integer i      -> "Integer: %d".formatted(i);
    case Long l         -> "Long: %d".formatted(l);
    case Double d       -> "Double: %.2f".formatted(d);
    case String s       -> "String: \"%s\"".formatted(s);
    case int[] arr      -> "int[]: length=%d".formatted(arr.length);
    case List<?> list   -> "List: size=%d".formatted(list.size());
    default             -> "Other: " + obj.getClass().getSimpleName();
};
```

### Guarded patterns
```java
Object value = 42;

String desc = switch (value) {
    case Integer i when i < 0   -> "Negative: " + i;
    case Integer i when i == 0  -> "Zero";
    case Integer i when i > 0 && i <= 100  -> "Small positive: " + i;
    case Integer i              -> "Large positive: " + i;
    case String s  when s.isBlank() -> "Blank string";
    case String s               -> "Non-blank string: " + s;
    default                     -> "Other";
};

System.out.println(desc); // Small positive: 42
```

### Exhaustive switch with sealed types (no default required)
```java
sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius)               implements Shape {}
record Rectangle(double width, double h)   implements Shape {}
record Triangle(double base, double height) implements Shape {}

double area(Shape shape) {
    return switch (shape) {
        case Circle c      -> Math.PI * c.radius() * c.radius();
        case Rectangle r   -> r.width() * r.h();
        case Triangle t    -> 0.5 * t.base() * t.height();
        // No default needed – all subtypes covered
    };
}
```

---

## 5. Record Patterns (Java 21 – Stable)

Record patterns allow destructuring records directly in patterns:

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}

Object obj = new Line(new Point(0, 0), new Point(3, 4));

// Destructure a record in instanceof
if (obj instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
    double len = Math.sqrt(Math.pow(x2-x1, 2) + Math.pow(y2-y1, 2));
    System.out.println("Line length: " + len); // Line length: 5.0
}

// Destructure in switch
String describe(Object shape) {
    return switch (shape) {
        case Point(int x, int y) when x == 0 && y == 0
            -> "Origin";
        case Point(int x, int y)
            -> "Point(%d, %d)".formatted(x, y);
        case Line(Point(int x1, int y1), Point(int x2, int y2))
            -> "Line from (%d,%d) to (%d,%d)".formatted(x1, y1, x2, y2);
        default
            -> "Unknown";
    };
}

System.out.println(describe(new Point(3, 4)));
// Point(3, 4)
System.out.println(describe(new Line(new Point(1, 2), new Point(4, 6))));
// Line from (1,2) to (4,6)
```

---

## 6. Text Blocks (Java 15 – Stable)

Text blocks provide multi-line string literals without escape characters:

```java
// Without text block
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello</p>\n" +
              "    </body>\n" +
              "</html>\n";

// With text block
String htmlBlock = """
        <html>
            <body>
                <p>Hello</p>
            </body>
        </html>
        """;

// JSON
String json = """
        {
            "name": "Alice",
            "age": 30,
            "city": "São Paulo"
        }
        """;

// SQL
String query = """
        SELECT u.name, u.email, COUNT(o.id) AS order_count
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE u.active = true
        GROUP BY u.id
        ORDER BY order_count DESC
        LIMIT 10
        """;

// HTML template
String name = "Alice";
String template = """
        Dear %s,
        
        Your account has been created.
        
        Regards,
        The Team
        """.formatted(name);
```

### Text block indentation rules
```java
// The closing """ determines the indentation level
// Spaces common to all lines are stripped
String indented = """
        line 1
        line 2
            indented line 3
        """;
// Result:
// "line 1\n"
// "line 2\n"
// "    indented line 3\n"

// Trailing newline – move closing """ to last content line to remove it
String noTrailingNewline = """
        no trailing newline""";
```

---

## 7. Unnamed Classes & Instance Main (Java 23 – Preview)

**JEP 463** (second preview in Java 23) allows writing simple programs without a class declaration:

```java
// Traditional HelloWorld
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```java
// Java 23 – unnamed class, instance main
// Save as Hello.java
void main() {
    System.out.println("Hello, World!");
}
```

```bash
java --enable-preview --source 23 Hello.java
```

### Using imports and other methods
```java
import java.util.List;

// Still an unnamed class
List<String> names = List.of("Alice", "Bob", "Carol");

void main() {
    greetAll(names);
}

void greetAll(List<String> names) {
    names.forEach(n -> System.out.println("Hello, " + n + "!"));
}
```

### Why this matters
This feature lowers the barrier to entry for beginners and makes Java more suitable for scripting and REPL-like workflows.

---

## 8. Primitive Types in Patterns (Java 23 – Preview)

**JEP 455** extends pattern matching to work with primitive types:

```java
// Without JEP 455 – primitives not allowed in patterns
Object obj = 42;
// case int i  -> ...  // Compile error before Java 23 preview

// With JEP 455 (--enable-preview)
Object value = 42;

String result = switch (value) {
    case int i when i < 0   -> "negative int: " + i;
    case int i              -> "positive int: " + i;
    case long l             -> "long: " + l;
    case double d           -> "double: " + d;
    default                 -> "other: " + value;
};
System.out.println(result); // positive int: 42

// instanceof with primitives
if (value instanceof int i) {
    System.out.println("It's an int: " + i);
}
```

To use preview features:
```bash
javac --enable-preview --release 23 MyClass.java
java  --enable-preview MyClass
```

---

## 9. String Templates (withdrawn – alternative approaches)

> **Note**: String templates (JEP 430 in Java 21, JEP 459 in Java 22) were **withdrawn** in Java 23. The feature is being redesigned.

Current alternatives:

```java
String name = "Alice";
int score = 95;

// String.format (classic)
String s1 = String.format("Player %s scored %d points.", name, score);

// String.formatted() (Java 15+) – cleaner syntax
String s2 = "Player %s scored %d points.".formatted(name, score);

// Text block with formatted()
String s3 = """
        Player: %s
        Score:  %d
        Grade:  %s
        """.formatted(name, score, score >= 90 ? "A" : "B");

// StringBuilder (when building piece by piece)
StringBuilder sb = new StringBuilder();
sb.append("Player ").append(name).append(" scored ").append(score).append(" points.");

// MessageFormat (for internalization)
String pattern = "On {0}, {1} scored {2} points.";
String result = java.text.MessageFormat.format(pattern,
    java.time.LocalDate.now(), name, score);
```

---

## 10. Other Notable Changes

### Stream Gatherers (JEP 473 – Second Preview in Java 23)
Gatherers extend the Stream API with custom intermediate operations:

```java
import java.util.stream.*;

// Built-in gatherers (from java.util.stream.Gatherers)
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Window (sliding/tumbling)
List<List<Integer>> windows = numbers.stream()
    .gather(Gatherers.windowSliding(3))   // [[1,2,3],[2,3,4],...,[8,9,10]]
    .toList();

List<List<Integer>> fixed = numbers.stream()
    .gather(Gatherers.windowFixed(3))     // [[1,2,3],[4,5,6],[7,8,9],[10]]
    .toList();

// Scan (like reduce, but keeps intermediate values)
List<Integer> runningSum = numbers.stream()
    .gather(Gatherers.scan(() -> 0, (sum, n) -> sum + n))
    .toList(); // [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]
```

### Scoped Values (JEP 464 – Second Preview)
Scoped values are immutable, thread-local-like values for structured concurrency:

```java
import java.lang.ScopedValue;

// Declare a scoped value
static final ScopedValue<String> CURRENT_USER = ScopedValue.newInstance();

void handleRequest(String user) {
    ScopedValue.where(CURRENT_USER, user).run(() -> {
        processOrder();  // can access CURRENT_USER within this scope
    });
}

void processOrder() {
    String user = CURRENT_USER.get(); // "alice"
    System.out.println("Processing order for: " + user);
}
```

### ZGC: Generational Mode by Default (JEP 474)
Java 23 makes Generational ZGC the default garbage collector mode:

```bash
# Before Java 23 – needed explicit flag
java -XX:+UseZGC -XX:+ZGenerational MyApp

# Java 23+ – generational ZGC is the default
java -XX:+UseZGC MyApp
```

### Markdown in Javadoc (JEP 467)
Write Javadoc using Markdown syntax:

```java
/**
 * Calculates the factorial of a number.
 *
 * ## Examples
 *
 * ```java
 * int result = factorial(5); // returns 120
 * ```
 *
 * ## Constraints
 * - `n` must be non-negative
 * - Result for `n > 20` overflows `long`
 *
 * @param n the input number
 * @return n!
 */
public static long factorial(int n) {
    if (n < 0) throw new IllegalArgumentException("n must be non-negative");
    return n == 0 ? 1 : n * factorial(n - 1);
}
```

---

## 11. Exercises

### Exercise 1 – Expression Evaluator
Using the sealed `Expr` interface from this module, add support for:
- `Expr.Sub(Expr left, Expr right)` – subtraction
- `Expr.Div(Expr left, Expr right)` – division (throw on divide by zero)
- `Expr.Pow(Expr base, Expr exp)` – exponentiation

Update `eval` and `prettyPrint` to handle all cases.

### Exercise 2 – Data Modeling with Records
Model a library system using records:
- `Book(String isbn, String title, String author, int year)`
- `Member(String id, String name, List<Book> borrowedBooks)`
- `Library(String name, List<Book> catalog, List<Member> members)`

Write methods using Stream API + pattern matching to:
1. Find all books by a given author
2. Find members who have overdue books (assume all borrowed > 30 days ago are overdue)
3. Get statistics: most borrowed book, most active member

### Exercise 3 – JSON-like Serializer
Using sealed types:
```java
sealed interface JsonValue permits JsonNull, JsonBool, JsonNumber, JsonString, JsonArray, JsonObject {}
```
Implement `String toJson(JsonValue value)` that serializes to JSON string using pattern matching.

### Exercise 4 – Unnamed Class
Write a small utility program using the **unnamed class** feature (Java 23 preview) that:
- Accepts a directory path as argument
- Lists all `.java` files recursively
- Prints their names sorted alphabetically

### Exercise 5 – Stream Gatherers
Using `Gatherers.windowSliding(n)`:
1. Find the maximum sliding average of 3 consecutive numbers in a list
2. Find the window of 3 numbers with the smallest sum

---

> ⬅️ Previous: [Module 12 – Concurrency & Virtual Threads](12-concurrency-and-virtual-threads.md) | ➡️ Next: [Module 14 – Advanced Topics](14-advanced-topics.md)
