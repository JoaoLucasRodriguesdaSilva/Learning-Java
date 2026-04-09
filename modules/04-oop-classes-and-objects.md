# Module 04 – Object-Oriented Programming: Classes & Objects

## Table of Contents
1. [OOP Principles Overview](#1-oop-principles-overview)
2. [Classes and Objects](#2-classes-and-objects)
3. [Constructors](#3-constructors)
4. [Instance vs Static Members](#4-instance-vs-static-members)
5. [Access Modifiers & Encapsulation](#5-access-modifiers--encapsulation)
6. [Getters and Setters](#6-getters-and-setters)
7. [The `this` Keyword](#7-the-this-keyword)
8. [Records (Java 16+)](#8-records-java-16)
9. [Object Methods: toString, equals, hashCode](#9-object-methods-tostring-equals-hashcode)
10. [Exercises](#10-exercises)

---

## 1. OOP Principles Overview

| Principle | Description |
|-----------|-------------|
| **Encapsulation** | Bundle data and behavior; hide implementation details |
| **Inheritance** | Derive new classes from existing ones (covered in Module 05) |
| **Polymorphism** | One interface, many implementations (Module 05) |
| **Abstraction** | Expose only relevant functionality (Module 06) |

---

## 2. Classes and Objects

A **class** is a blueprint. An **object** is an instance of a class.

```java
// Class definition
public class Car {
    // Fields (instance variables)
    String brand;
    String model;
    int    year;
    double fuelLevel;

    // Method
    void drive() {
        System.out.println(brand + " " + model + " is driving.");
    }

    void refuel(double liters) {
        fuelLevel += liters;
        System.out.println("Refueled. Fuel level: " + fuelLevel + "L");
    }
}
```

```java
// Creating objects
Car car1 = new Car();
car1.brand = "Toyota";
car1.model = "Corolla";
car1.year  = 2023;

Car car2 = new Car();
car2.brand = "Honda";
car2.model = "Civic";

car1.drive();    // Toyota Corolla is driving.
car2.drive();    // Honda Civic is driving.

// Each object has its own copy of instance variables
System.out.println(car1 == car2); // false – different objects
```

---

## 3. Constructors

A **constructor** initializes an object when it's created with `new`.

```java
public class Person {
    String name;
    int age;

    // Default constructor (no parameters)
    public Person() {
        this.name = "Unknown";
        this.age  = 0;
    }

    // Parameterized constructor
    public Person(String name, int age) {
        this.name = name;
        this.age  = age;
    }

    // Constructor chaining with this()
    public Person(String name) {
        this(name, 0);   // calls the two-parameter constructor
    }
}
```

```java
Person p1 = new Person();              // Unknown, 0
Person p2 = new Person("Alice", 30);   // Alice, 30
Person p3 = new Person("Bob");         // Bob, 0
```

> If you define **no** constructor, Java provides a default (no-arg) constructor automatically.  
> If you define **any** constructor, the automatic default is **not** provided.

---

## 4. Instance vs Static Members

| | Instance | Static |
|--|----------|--------|
| **Belongs to** | Each object | The class itself |
| **Access** | Via object reference | Via class name |
| **Memory** | Per object | Once (shared) |

```java
public class Counter {
    int count;               // instance variable – unique per object
    static int totalCreated; // class (static) variable – shared

    public Counter() {
        totalCreated++;
    }

    void increment() {
        count++;
    }

    static int getTotal() {  // static method
        return totalCreated;
    }
}
```

```java
Counter c1 = new Counter();
Counter c2 = new Counter();

c1.increment();
c1.increment();
c2.increment();

System.out.println(c1.count);           // 2
System.out.println(c2.count);           // 1
System.out.println(Counter.totalCreated); // 2  (accessed via class name)
System.out.println(Counter.getTotal());   // 2
```

### Static initializer block
```java
public class Config {
    static final String VERSION;

    static {
        VERSION = "1.0.0";  // runs once when class is first loaded
        System.out.println("Config initialized");
    }
}
```

---

## 5. Access Modifiers & Encapsulation

| Modifier | Same Class | Same Package | Subclass | Anywhere |
|----------|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Encapsulation** = `private` fields + `public` getter/setter methods:

```java
public class BankAccount {
    private double balance;   // hidden from outside
    private String owner;

    public BankAccount(String owner, double initialBalance) {
        this.owner   = owner;
        this.balance = initialBalance;
    }

    // Controlled access
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        } else {
            System.out.println("Invalid withdrawal amount.");
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

---

## 6. Getters and Setters

```java
public class Product {
    private String name;
    private double price;

    public Product(String name, double price) {
        this.name  = name;
        setPrice(price);   // use setter for validation
    }

    // Getter
    public String getName() { return name; }

    // Setter with validation
    public void setPrice(double price) {
        if (price < 0) throw new IllegalArgumentException("Price cannot be negative");
        this.price = price;
    }

    public double getPrice() { return price; }
}
```

```java
Product p = new Product("Laptop", 999.99);
System.out.println(p.getName());    // Laptop
System.out.println(p.getPrice());   // 999.99
p.setPrice(1099.99);
// p.price = -5;  // Compile error – private field
```

---

## 7. The `this` Keyword

```java
public class Rectangle {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width  = width;   // 'this' disambiguates field vs parameter
        this.height = height;
    }

    public Rectangle scale(double factor) {
        return new Rectangle(this.width * factor, this.height * factor);
    }

    // Method chaining with 'this'
    public Rectangle setWidth(double width) {
        this.width = width;
        return this;   // return current object to allow chaining
    }

    public Rectangle setHeight(double height) {
        this.height = height;
        return this;
    }

    public double area() {
        return width * height;
    }
}
```

```java
Rectangle r = new Rectangle(0, 0)
                  .setWidth(5)
                  .setHeight(3);
System.out.println(r.area()); // 15.0
```

---

## 8. Records (Java 16+)

A **record** is a concise, immutable data carrier. Java automatically generates constructor, getters, `equals`, `hashCode`, and `toString`.

```java
// Traditional class
public class PointOld {
    private final int x;
    private final int y;

    public PointOld(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public int x() { return x; }
    public int y() { return y; }
    // ... equals, hashCode, toString ...
}

// Equivalent record (1 line!)
public record Point(int x, int y) {}
```

```java
Point p = new Point(3, 4);
System.out.println(p.x());        // 3  (getter)
System.out.println(p.y());        // 4
System.out.println(p);            // Point[x=3, y=4]

Point p2 = new Point(3, 4);
System.out.println(p.equals(p2)); // true (structural equality)
```

### Compact constructor for validation
```java
public record Temperature(double celsius) {
    public Temperature {   // compact constructor
        if (celsius < -273.15) {
            throw new IllegalArgumentException("Below absolute zero!");
        }
    }
}
```

### Custom methods on records
```java
public record Circle(double radius) {
    public double area() {
        return Math.PI * radius * radius;
    }
    public double circumference() {
        return 2 * Math.PI * radius;
    }
}
```

---

## 9. Object Methods: toString, equals, hashCode

All Java classes inherit from `Object`. The three most important methods to override:

```java
public class Book {
    private String title;
    private String author;
    private int year;

    public Book(String title, String author, int year) {
        this.title  = title;
        this.author = author;
        this.year   = year;
    }

    @Override
    public String toString() {
        return "Book{title='" + title + "', author='" + author + "', year=" + year + "}";
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;              // same object
        if (!(obj instanceof Book other)) return false;  // pattern matching instanceof
        return year == other.year
            && title.equals(other.title)
            && author.equals(other.author);
    }

    @Override
    public int hashCode() {
        return java.util.Objects.hash(title, author, year);
    }
}
```

```java
Book b1 = new Book("Clean Code", "Robert Martin", 2008);
Book b2 = new Book("Clean Code", "Robert Martin", 2008);

System.out.println(b1);            // Book{title='Clean Code', author='Robert Martin', year=2008}
System.out.println(b1.equals(b2)); // true  (content-based)
System.out.println(b1 == b2);      // false (different references)

// HashMap and HashSet rely on equals + hashCode being consistent
java.util.Set<Book> library = new java.util.HashSet<>();
library.add(b1);
library.add(b2);
System.out.println(library.size()); // 1 (treated as same book)
```

---

## 10. Exercises

### Exercise 1 – Student Class
Create a `Student` class with:
- Private fields: `name` (String), `id` (int), `gpa` (double)
- Constructor with all fields
- Getters and setters (validate GPA is 0.0–4.0)
- Override `toString()` to print: `Student[id=1, name=Alice, gpa=3.7]`

### Exercise 2 – Stack Implementation
Create a `Stack` class that:
- Stores integers in a private array (max 10 elements)
- Has `push(int)`, `pop()`, `peek()`, `isEmpty()`, `isFull()` methods
- Throws `RuntimeException` on overflow/underflow

### Exercise 3 – Bank Account
Extend the `BankAccount` class to:
- Track all transactions in a `List<String>` (e.g., "+200.00", "-50.00")
- Add a `printStatement()` method that lists all transactions and the final balance

### Exercise 4 – Record Conversion
Take this class and convert it to a record:
```java
public class Coordinates {
    private final double latitude;
    private final double longitude;
    // constructor, getters, equals, hashCode, toString ...
}
```
Add a method `distanceTo(Coordinates other)` that returns the Euclidean distance.

### Exercise 5 – Singleton Pattern
Implement the Singleton design pattern: a class that can only have one instance.
```java
public class AppConfig {
    private static AppConfig instance;
    private String dbUrl = "jdbc:postgresql://localhost/mydb";

    private AppConfig() {}

    public static AppConfig getInstance() {
        // TODO: return the single instance, creating it if needed
    }

    public String getDbUrl() { return dbUrl; }
}
```

---

> ⬅️ Previous: [Module 03 – Control Flow](03-control-flow.md) | ➡️ Next: [Module 05 – Inheritance & Polymorphism](05-inheritance-and-polymorphism.md)
