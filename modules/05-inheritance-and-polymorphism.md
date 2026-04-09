# Module 05 – Inheritance & Polymorphism

## Table of Contents
1. [Inheritance Basics](#1-inheritance-basics)
2. [Method Overriding](#2-method-overriding)
3. [The `super` Keyword](#3-the-super-keyword)
4. [Constructors and Inheritance](#4-constructors-and-inheritance)
5. [The `Object` Class](#5-the-object-class)
6. [Polymorphism](#6-polymorphism)
7. [`instanceof` and Pattern Matching](#7-instanceof-and-pattern-matching)
8. [Abstract Classes](#8-abstract-classes)
9. [`final` Classes and Methods](#9-final-classes-and-methods)
10. [Exercises](#10-exercises)

---

## 1. Inheritance Basics

Inheritance allows a class (**subclass/child**) to extend another class (**superclass/parent**), inheriting its fields and methods.

```java
// Superclass
public class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age  = age;
    }

    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }
}

// Subclass
public class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);      // call parent constructor
        this.breed = breed;
    }

    public void bark() {
        System.out.println(name + " says: Woof!");
    }
}
```

```java
Dog dog = new Dog("Rex", 3, "Labrador");
dog.eat();   // inherited – Rex is eating.
dog.sleep(); // inherited – Rex is sleeping.
dog.bark();  // own method – Rex says: Woof!
```

> Java supports **single inheritance** for classes (a class can extend only one class).  
> Multiple inheritance is achieved through **interfaces** (Module 06).

---

## 2. Method Overriding

A subclass can **override** a parent method to provide a different implementation:

```java
public class Animal {
    public String sound() {
        return "...";
    }

    public void describe() {
        System.out.println("I am " + name + " and I say: " + sound());
    }
}

public class Dog extends Animal {
    @Override          // annotation – recommended for clarity
    public String sound() {
        return "Woof";
    }
}

public class Cat extends Animal {
    @Override
    public String sound() {
        return "Meow";
    }
}
```

```java
Animal a = new Dog("Rex", 3, "Lab");
a.describe();  // I am Rex and I say: Woof
```

### Rules for overriding
1. Same method name and parameter list
2. Return type must be the same or a **covariant** (subtype)
3. Access modifier must be **same or more permissive**
4. Cannot override `static`, `private`, or `final` methods

---

## 3. The `super` Keyword

`super` refers to the **immediate parent class**:

```java
public class Vehicle {
    protected int speed;

    public Vehicle(int speed) {
        this.speed = speed;
    }

    public String describe() {
        return "Vehicle at speed " + speed;
    }
}

public class Car extends Vehicle {
    private String brand;

    public Car(String brand, int speed) {
        super(speed);         // call parent constructor
        this.brand = brand;
    }

    @Override
    public String describe() {
        return super.describe() + " | Brand: " + brand; // call parent method
    }
}
```

```java
Car car = new Car("Toyota", 120);
System.out.println(car.describe());
// Vehicle at speed 120 | Brand: Toyota
```

---

## 4. Constructors and Inheritance

- `super()` must be the **first statement** in a subclass constructor
- If not explicit, Java inserts `super()` automatically (calls the no-arg parent constructor)
- If the parent has no no-arg constructor, you **must** call an appropriate `super(...)` explicitly

```java
public class Shape {
    private String color;

    public Shape(String color) {
        this.color = color;
        System.out.println("Shape created: " + color);
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);   // must be first line
        this.radius = radius;
        System.out.println("Circle created with radius " + radius);
    }
}
```

```java
Circle c = new Circle("Red", 5.0);
// Output:
// Shape created: Red
// Circle created with radius 5.0
```

---

## 5. The `Object` Class

Every Java class implicitly extends `java.lang.Object`. Its key methods:

| Method | Description |
|--------|-------------|
| `toString()` | String representation |
| `equals(Object)` | Structural equality |
| `hashCode()` | Hash code for use in hash-based collections |
| `getClass()` | Returns the runtime class |
| `clone()` | Creates a copy (requires `Cloneable`) |
| `finalize()` | Called before GC (deprecated in Java 9) |

---

## 6. Polymorphism

Polymorphism = **"many forms"**. A parent reference can refer to any subclass object.

```java
Animal[] animals = {
    new Dog("Rex",   3, "Lab"),
    new Cat("Whiskers", 5),
    new Dog("Buddy", 2, "Poodle")
};

for (Animal a : animals) {
    a.describe();   // calls each class's overridden method
}
// I am Rex and I say: Woof
// I am Whiskers and I say: Meow
// I am Buddy and I say: Woof
```

### Compile-time vs Runtime polymorphism

| | Compile-time (Static) | Runtime (Dynamic) |
|--|----------------------|-------------------|
| Also called | Method overloading | Method overriding |
| Resolved at | Compile time | Runtime |
| Example | `add(int)` vs `add(double)` | `animal.sound()` |

```java
// Overloading (compile-time)
public class Printer {
    public void print(int n)    { System.out.println("int: " + n); }
    public void print(double d) { System.out.println("double: " + d); }
    public void print(String s) { System.out.println("String: " + s); }
}

Printer p = new Printer();
p.print(42);     // int: 42
p.print(3.14);   // double: 3.14
p.print("Hi");   // String: Hi
```

---

## 7. `instanceof` and Pattern Matching

`instanceof` checks whether an object is of a specific type:

```java
// Classic style (before Java 16)
Animal a = new Dog("Rex", 3, "Lab");
if (a instanceof Dog) {
    Dog d = (Dog) a;   // explicit cast needed
    d.bark();
}

// Pattern matching instanceof (Java 16+)
if (a instanceof Dog d) {       // declares and casts in one step
    d.bark();
}

// Pattern matching with condition (Java 21+)
if (a instanceof Dog d && d.getBreed().equals("Labrador")) {
    System.out.println("It's a Labrador!");
}
```

---

## 8. Abstract Classes

An **abstract class** cannot be instantiated and may contain abstract methods (no body) that subclasses must implement:

```java
public abstract class Shape {
    private String color;

    public Shape(String color) {
        this.color = color;
    }

    // Abstract method – no body
    public abstract double area();
    public abstract double perimeter();

    // Concrete method – shared implementation
    public void printInfo() {
        System.out.printf("Shape: %s | Color: %s | Area: %.2f%n",
            getClass().getSimpleName(), color, area());
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }

    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}

public class Rectangle extends Shape {
    private double width, height;

    public Rectangle(String color, double width, double height) {
        super(color);
        this.width  = width;
        this.height = height;
    }

    @Override public double area()      { return width * height; }
    @Override public double perimeter() { return 2 * (width + height); }
}
```

```java
Shape[] shapes = {
    new Circle("Red", 5),
    new Rectangle("Blue", 4, 6)
};

for (Shape s : shapes) {
    s.printInfo();
}
// Shape: Circle    | Color: Red  | Area: 78.54
// Shape: Rectangle | Color: Blue | Area: 24.00
```

---

## 9. `final` Classes and Methods

```java
// final class – cannot be subclassed
public final class ImmutablePoint {
    private final int x, y;
    public ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
    public int x() { return x; }
    public int y() { return y; }
}

// class ExtendedPoint extends ImmutablePoint {}  // Compile error!

// final method – cannot be overridden
public class Base {
    public final void criticalOperation() {
        System.out.println("This cannot be overridden.");
    }
}
```

`String`, `Integer`, and many Java library classes are `final` for security and immutability.

---

## 10. Exercises

### Exercise 1 – Shape Hierarchy
Create an abstract class `Shape` with abstract methods `area()` and `perimeter()`. Implement:
- `Circle(double radius)`
- `Rectangle(double width, double height)`
- `Triangle(double a, double b, double c)` using Heron's formula

Write a method that accepts `Shape[]` and prints the total area.

### Exercise 2 – Employee Hierarchy
Create:
- `Employee` (name, salary) with `getSalary()` and `toString()`
- `Manager extends Employee` with a `bonus` field; overrides `getSalary()` to include bonus
- `Intern extends Employee` with a fixed 20% reduction; overrides `getSalary()`

Demonstrate polymorphism by storing all in an `Employee[]`.

### Exercise 3 – Overloading vs Overriding
Create a class `Calculator` with overloaded `add` methods:
- `add(int, int)`
- `add(double, double)`
- `add(int, int, int)`

Then create `ScientificCalculator extends Calculator` that overrides `add(double, double)` to first round inputs to 2 decimal places.

### Exercise 4 – `instanceof` Dispatch
Given an `Object[] items` containing a mix of `String`, `Integer`, `Double`, `Boolean`, and `null` values, use pattern matching to print each element's type and its value doubled (for numbers) or uppercased (for strings).

### Exercise 5 – Covariant Return Types
```java
class Animal {
    public Animal create() { return new Animal(); }
}
```
Create `Dog extends Animal` that overrides `create()` returning a `Dog` instead. Demonstrate that this is a valid covariant override.

---

> ⬅️ Previous: [Module 04 – OOP: Classes & Objects](04-oop-classes-and-objects.md) | ➡️ Next: [Module 06 – Interfaces & Abstract Classes](06-interfaces-and-abstract-classes.md)
