# Module 06 – Interfaces & Abstract Classes

## Table of Contents
1. [Interfaces](#1-interfaces)
2. [Implementing Multiple Interfaces](#2-implementing-multiple-interfaces)
3. [Default Methods](#3-default-methods)
4. [Static Methods in Interfaces](#4-static-methods-in-interfaces)
5. [Private Methods in Interfaces (Java 9+)](#5-private-methods-in-interfaces-java-9)
6. [Functional Interfaces](#6-functional-interfaces)
7. [Sealed Interfaces (Java 17+)](#7-sealed-interfaces-java-17)
8. [Interface vs Abstract Class](#8-interface-vs-abstract-class)
9. [Common Java Interfaces](#9-common-java-interfaces)
10. [Exercises](#10-exercises)

---

## 1. Interfaces

An **interface** defines a **contract** – a set of methods that implementing classes must provide.

```java
public interface Drawable {
    // Abstract methods (implicitly public abstract)
    void draw();
    void resize(double factor);

    // Constant (implicitly public static final)
    double DEFAULT_SCALE = 1.0;
}

public interface Colorable {
    void setColor(String color);
    String getColor();
}
```

```java
// A class implements one or more interfaces
public class Circle implements Drawable, Colorable {
    private double radius;
    private String color;

    public Circle(double radius) {
        this.radius = radius;
        this.color  = "black";
    }

    @Override
    public void draw() {
        System.out.println("Drawing circle with radius " + radius + " in " + color);
    }

    @Override
    public void resize(double factor) {
        radius *= factor;
    }

    @Override
    public void setColor(String color) { this.color = color; }

    @Override
    public String getColor() { return color; }
}
```

```java
Circle c = new Circle(5.0);
c.setColor("red");
c.draw();           // Drawing circle with radius 5.0 in red
c.resize(2.0);
c.draw();           // Drawing circle with radius 10.0 in red

// Interface as type
Drawable d = new Circle(3.0);
d.draw();
// d.setColor("blue");  // Compile error – Drawable doesn't have setColor
```

---

## 2. Implementing Multiple Interfaces

A class can implement as many interfaces as needed:

```java
public interface Flyable {
    void fly();
    default double getAltitude() { return 1000.0; }
}

public interface Swimmable {
    void swim();
}

public class Duck extends Animal implements Flyable, Swimmable {
    @Override public void fly()  { System.out.println(name + " is flying!"); }
    @Override public void swim() { System.out.println(name + " is swimming!"); }
}
```

---

## 3. Default Methods

**Default methods** provide a concrete implementation in an interface, enabling backward-compatible API evolution:

```java
public interface Collection<E> {
    void add(E element);
    void remove(E element);
    int size();

    // Default method – classes don't have to override this
    default boolean isEmpty() {
        return size() == 0;
    }

    default void addAll(java.util.List<E> elements) {
        for (E e : elements) {
            add(e);
        }
    }
}
```

### Diamond problem resolution
If two interfaces provide a default method with the same signature, the implementing class must override it:

```java
interface A {
    default String hello() { return "Hello from A"; }
}

interface B {
    default String hello() { return "Hello from B"; }
}

class C implements A, B {
    @Override
    public String hello() {
        return A.super.hello() + " + " + B.super.hello(); // explicit
    }
}
```

---

## 4. Static Methods in Interfaces

Static interface methods belong to the interface itself, not implementing classes:

```java
public interface MathOperation {
    int operate(int a, int b);

    // Static factory methods
    static MathOperation add()      { return (a, b) -> a + b; }
    static MathOperation subtract() { return (a, b) -> a - b; }
    static MathOperation multiply() { return (a, b) -> a * b; }
}
```

```java
MathOperation add = MathOperation.add();
System.out.println(add.operate(5, 3)); // 8

MathOperation mul = MathOperation.multiply();
System.out.println(mul.operate(5, 3)); // 15
```

---

## 5. Private Methods in Interfaces (Java 9+)

Private methods eliminate code duplication within interface default methods:

```java
public interface Logger {
    void log(String message);

    default void logInfo(String message) {
        log(format("INFO", message));
    }

    default void logError(String message) {
        log(format("ERROR", message));
    }

    // Private helper – not part of the contract
    private String format(String level, String message) {
        return "[" + level + "] " + java.time.LocalTime.now() + " - " + message;
    }
}
```

---

## 6. Functional Interfaces

A **functional interface** has exactly one abstract method. It can be implemented with a **lambda expression**.

```java
@FunctionalInterface
public interface Validator<T> {
    boolean validate(T value);

    // May have default/static methods
    default Validator<T> and(Validator<T> other) {
        return value -> this.validate(value) && other.validate(value);
    }
}
```

```java
Validator<String> notEmpty  = s -> !s.isEmpty();
Validator<String> notTooLong = s -> s.length() <= 50;

Validator<String> combined = notEmpty.and(notTooLong);

System.out.println(combined.validate("Hello"));   // true
System.out.println(combined.validate(""));        // false
```

### Built-in functional interfaces in `java.util.function`

| Interface | Abstract Method | Use Case |
|-----------|----------------|----------|
| `Supplier<T>` | `T get()` | Provides a value |
| `Consumer<T>` | `void accept(T)` | Consumes a value |
| `Function<T,R>` | `R apply(T)` | Transforms a value |
| `Predicate<T>` | `boolean test(T)` | Tests a condition |
| `BiFunction<T,U,R>` | `R apply(T, U)` | Two inputs, one output |
| `UnaryOperator<T>` | `T apply(T)` | Same type in and out |

```java
import java.util.function.*;

Predicate<Integer> isEven = n -> n % 2 == 0;
Function<String, Integer> length = String::length;
Consumer<String> printer = System.out::println;
Supplier<Double> random = Math::random;

System.out.println(isEven.test(4));        // true
System.out.println(length.apply("Java"));  // 4
printer.accept("Hello");                   // Hello
System.out.println(random.get());          // e.g. 0.7341
```

---

## 7. Sealed Interfaces (Java 17+)

**Sealed interfaces** restrict which classes can implement them, making the type hierarchy explicit and enabling exhaustive pattern matching:

```java
public sealed interface Shape
    permits Circle, Rectangle, Triangle {}

public record Circle(double radius)           implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base, double height)   implements Shape {}
```

```java
// Exhaustive switch – no default needed because all subtypes are known
Shape shape = new Circle(5.0);

double area = switch (shape) {
    case Circle c         -> Math.PI * c.radius() * c.radius();
    case Rectangle r      -> r.width() * r.height();
    case Triangle t       -> 0.5 * t.base() * t.height();
};

System.out.printf("Area: %.2f%n", area); // Area: 78.54
```

### Permitted subtype options

| Modifier | Meaning |
|----------|---------|
| `final` | No further extension |
| `sealed` | Must specify its own permits |
| `non-sealed` | Open for further extension |

---

## 8. Interface vs Abstract Class

| Feature | Interface | Abstract Class |
|---------|-----------|----------------|
| Multiple inheritance | ✅ (multiple) | ❌ (single) |
| Fields | Constants only | Any fields |
| Constructors | ❌ | ✅ |
| Default methods | ✅ (Java 8+) | ✅ |
| Access modifiers for methods | public (implicitly) | Any |
| Use when | Defining a capability/role | Sharing code among related classes |

### When to use each

```
Use an interface when you want to define WHAT something can do
(Printable, Serializable, Comparable)

Use an abstract class when you want to provide a PARTIAL implementation
for a group of related classes (Animal → Dog, Cat, Bird)
```

---

## 9. Common Java Interfaces

### `Comparable<T>` – natural ordering
```java
public class Student implements Comparable<Student> {
    private String name;
    private double gpa;

    public Student(String name, double gpa) {
        this.name = name;
        this.gpa  = gpa;
    }

    @Override
    public int compareTo(Student other) {
        return Double.compare(other.gpa, this.gpa); // descending by GPA
    }

    @Override
    public String toString() { return name + " (" + gpa + ")"; }
}
```

```java
import java.util.*;
List<Student> students = new ArrayList<>(List.of(
    new Student("Alice", 3.8),
    new Student("Bob",   3.5),
    new Student("Carol", 3.9)
));
Collections.sort(students);
students.forEach(System.out::println);
// Carol (3.9)
// Alice (3.8)
// Bob (3.5)
```

### `Iterable<T>` – for-each compatibility
```java
public class NumberRange implements Iterable<Integer> {
    private final int start, end;

    public NumberRange(int start, int end) {
        this.start = start;
        this.end   = end;
    }

    @Override
    public java.util.Iterator<Integer> iterator() {
        return new java.util.Iterator<>() {
            int current = start;
            public boolean hasNext() { return current <= end; }
            public Integer next()    { return current++; }
        };
    }
}
```

```java
for (int n : new NumberRange(1, 5)) {
    System.out.print(n + " "); // 1 2 3 4 5
}
```

---

## 10. Exercises

### Exercise 1 – Payment System
Define a `PaymentMethod` interface with:
- `void pay(double amount)`
- `String getDescription()`
- `default void payWithReceipt(double amount)` that calls `pay` and prints a receipt

Implement: `CreditCard`, `PayPal`, `CryptoCurrency`.

### Exercise 2 – Sealed Shape Hierarchy
Create a sealed interface `Shape` with permits `Circle`, `Square`, `Triangle` (all as records). Write a method `describe(Shape s)` that uses an exhaustive switch and returns a description string.

### Exercise 3 – Event System
Design an `EventListener<T>` functional interface with `void onEvent(T event)`. Create an `EventBus<T>` class that holds a list of listeners and broadcasts events to all of them. Demo with `String` events.

### Exercise 4 – Comparator
Sort a list of products by price (ascending), then by name (alphabetically) as a tiebreaker using `Comparator.comparing().thenComparing()`.

### Exercise 5 – Plugin Architecture
Design an interface `Plugin` with methods `String getName()`, `void execute()`. Create a `PluginRegistry` that stores plugins by name and can run them by name. Implement two plugins: `LogPlugin` and `MetricsPlugin`.

---

> ⬅️ Previous: [Module 05 – Inheritance & Polymorphism](05-inheritance-and-polymorphism.md) | ➡️ Next: [Module 07 – Collections Framework](07-collections-framework.md)
