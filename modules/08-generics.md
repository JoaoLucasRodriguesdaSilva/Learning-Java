# Module 08 – Generics

## Table of Contents
1. [Why Generics?](#1-why-generics)
2. [Generic Classes](#2-generic-classes)
3. [Generic Methods](#3-generic-methods)
4. [Bounded Type Parameters](#4-bounded-type-parameters)
5. [Wildcards](#5-wildcards)
6. [Type Erasure](#6-type-erasure)
7. [Generic Interfaces](#7-generic-interfaces)
8. [Common Generic Patterns](#8-common-generic-patterns)
9. [Exercises](#9-exercises)

---

## 1. Why Generics?

Without generics you'd lose **type safety** and need explicit casts:

```java
// Without generics (legacy code)
List list = new ArrayList();
list.add("hello");
list.add(42);         // accidentally added an int!
String s = (String) list.get(1); // ClassCastException at runtime!

// With generics
List<String> safeList = new ArrayList<>();
safeList.add("hello");
// safeList.add(42);  // Compile error – caught early!
String s = safeList.get(0);  // no cast needed
```

Generics provide:
- **Type safety** – errors caught at compile time
- **Code reuse** – write algorithm once, use with any type
- **Readability** – intent is clear from the code

---

## 2. Generic Classes

```java
// T is a type parameter (convention: T = Type, E = Element, K = Key, V = Value, N = Number)
public class Box<T> {
    private T content;

    public Box(T content) {
        this.content = content;
    }

    public T get() {
        return content;
    }

    public void set(T content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Box[" + content + "]";
    }
}
```

```java
Box<String>  stringBox = new Box<>("Hello");
Box<Integer> intBox    = new Box<>(42);
Box<Double>  dblBox    = new Box<>(3.14);

System.out.println(stringBox.get());  // Hello
System.out.println(intBox.get());     // 42

// Diamond operator – type inferred from left side (Java 7+)
Box<String> inferred = new Box<>("World");
```

### Generic class with multiple type parameters
```java
public class Pair<A, B> {
    private final A first;
    private final B second;

    public Pair(A first, B second) {
        this.first  = first;
        this.second = second;
    }

    public A first()  { return first; }
    public B second() { return second; }

    @Override
    public String toString() {
        return "(" + first + ", " + second + ")";
    }

    public static <A, B> Pair<A, B> of(A a, B b) {
        return new Pair<>(a, b);
    }
}
```

```java
Pair<String, Integer> entry = Pair.of("Alice", 30);
System.out.println(entry);               // (Alice, 30)
System.out.println(entry.first());       // Alice
System.out.println(entry.second());      // 30
```

---

## 3. Generic Methods

A method can declare its own type parameters, independent of the class:

```java
public class ArrayUtils {

    // Generic method – T is declared before the return type
    public static <T> void swap(T[] array, int i, int j) {
        T temp   = array[i];
        array[i] = array[j];
        array[j] = temp;
    }

    public static <T> T[] reverse(T[] array) {
        for (int i = 0, j = array.length - 1; i < j; i++, j--) {
            swap(array, i, j);
        }
        return array;
    }

    public static <T> java.util.Optional<T> findFirst(
            T[] array, java.util.function.Predicate<T> predicate) {
        for (T element : array) {
            if (predicate.test(element)) {
                return java.util.Optional.of(element);
            }
        }
        return java.util.Optional.empty();
    }
}
```

```java
String[] names = {"Alice", "Bob", "Carol"};
ArrayUtils.swap(names, 0, 2);
System.out.println(java.util.Arrays.toString(names)); // [Carol, Bob, Alice]

ArrayUtils.reverse(names);
System.out.println(java.util.Arrays.toString(names)); // [Alice, Bob, Carol]

var found = ArrayUtils.findFirst(names, s -> s.startsWith("B"));
found.ifPresent(System.out::println); // Bob
```

---

## 4. Bounded Type Parameters

Restrict which types can be used as type arguments.

### Upper bound – `<T extends SomeType>`
```java
public class NumberBox<T extends Number> {
    private T value;

    public NumberBox(T value) {
        this.value = value;
    }

    // Safe – T is guaranteed to be a Number
    public double doubleValue() {
        return value.doubleValue();
    }

    public T getValue() { return value; }
}
```

```java
NumberBox<Integer> intBox = new NumberBox<>(42);
NumberBox<Double>  dblBox = new NumberBox<>(3.14);
// NumberBox<String>  strBox = new NumberBox<>("hi"); // Compile error!

System.out.println(intBox.doubleValue()); // 42.0
```

### Multiple bounds
```java
// T must extend Comparable AND implement Serializable
public <T extends Comparable<T> & java.io.Serializable> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}
```

### Generic method with bounds – finding the sum
```java
public static <T extends Number> double sum(java.util.List<T> list) {
    double total = 0;
    for (T n : list) {
        total += n.doubleValue();
    }
    return total;
}
```

```java
System.out.println(sum(List.of(1, 2, 3)));           // 6.0
System.out.println(sum(List.of(1.5, 2.5, 3.0)));     // 7.0
```

---

## 5. Wildcards

Wildcards (`?`) represent unknown types in generic types.

### Unbounded wildcard – `<?>`
```java
// Can accept List<String>, List<Integer>, etc.
public static void printAll(java.util.List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}
```

### Upper-bounded wildcard – `<? extends T>`
Read-only ("producer"): accepts `T` or any subtype

```java
// PECS: Producer Extends
public static double sumList(java.util.List<? extends Number> list) {
    double sum = 0;
    for (Number n : list) {
        sum += n.doubleValue();
    }
    return sum;
}
```

```java
List<Integer> ints    = List.of(1, 2, 3);
List<Double>  doubles = List.of(1.5, 2.5);

System.out.println(sumList(ints));    // 6.0
System.out.println(sumList(doubles)); // 4.0
```

### Lower-bounded wildcard – `<? super T>`
Write-only ("consumer"): accepts `T` or any supertype

```java
// PECS: Consumer Super
public static void addNumbers(java.util.List<? super Integer> list) {
    for (int i = 1; i <= 5; i++) {
        list.add(i);  // safe – we know it accepts Integer or higher
    }
}
```

```java
List<Number> numbers = new ArrayList<>();
addNumbers(numbers);
System.out.println(numbers); // [1, 2, 3, 4, 5]
```

### PECS Rule
> **P**roducer → **E**xtends, **C**onsumer → **S**uper

```java
// Copying elements: src produces (extends), dest consumes (super)
public static <T> void copy(
        java.util.List<? extends T> src,
        java.util.List<? super T>   dest) {
    for (T item : src) {
        dest.add(item);
    }
}
```

---

## 6. Type Erasure

Generics are a **compile-time** feature. At runtime, type information is **erased** – all `List<String>` and `List<Integer>` become just `List`.

```java
List<String>  strings  = new ArrayList<>();
List<Integer> integers = new ArrayList<>();

// At runtime, both are just java.util.ArrayList
System.out.println(strings.getClass() == integers.getClass()); // true

// You CANNOT do:
// T obj = new T();            // Cannot instantiate type parameter
// T[] arr = new T[10];        // Cannot create generic array
// if (list instanceof List<String>) {}  // Cannot use parameterized type in instanceof
```

### Reifiable types workaround
```java
public class TypeSafeList<T> {
    private final Class<T> type;
    private final List<T> list = new ArrayList<>();

    public TypeSafeList(Class<T> type) {
        this.type = type;
    }

    public void add(Object item) {
        list.add(type.cast(item)); // ClassCastException if wrong type
    }
}
```

---

## 7. Generic Interfaces

```java
public interface Repository<T, ID> {
    void save(T entity);
    java.util.Optional<T> findById(ID id);
    java.util.List<T> findAll();
    void delete(ID id);
}

public record User(int id, String name) {}

public class InMemoryUserRepository implements Repository<User, Integer> {
    private final java.util.Map<Integer, User> store = new java.util.HashMap<>();

    @Override
    public void save(User user) {
        store.put(user.id(), user);
    }

    @Override
    public java.util.Optional<User> findById(Integer id) {
        return java.util.Optional.ofNullable(store.get(id));
    }

    @Override
    public java.util.List<User> findAll() {
        return new java.util.ArrayList<>(store.values());
    }

    @Override
    public void delete(Integer id) {
        store.remove(id);
    }
}
```

```java
Repository<User, Integer> repo = new InMemoryUserRepository();
repo.save(new User(1, "Alice"));
repo.save(new User(2, "Bob"));

repo.findById(1).ifPresent(System.out::println); // User[id=1, name=Alice]
System.out.println(repo.findAll());
```

---

## 8. Common Generic Patterns

### Generic Stack
```java
public class Stack<T> {
    private final java.util.ArrayDeque<T> deque = new java.util.ArrayDeque<>();

    public void push(T item)   { deque.addFirst(item); }
    public T    pop()          { return deque.removeFirst(); }
    public T    peek()         { return deque.peekFirst(); }
    public boolean isEmpty()   { return deque.isEmpty(); }
    public int  size()         { return deque.size(); }

    @Override
    public String toString()   { return deque.toString(); }
}
```

### Result type (Either pattern)
```java
public sealed interface Result<T>
    permits Result.Success, Result.Failure {

    record Success<T>(T value) implements Result<T> {}
    record Failure<T>(String error) implements Result<T> {}

    static <T> Result<T> success(T value)   { return new Success<>(value); }
    static <T> Result<T> failure(String msg){ return new Failure<>(msg); }

    default boolean isSuccess() { return this instanceof Success; }
}
```

```java
Result<Integer> result = divide(10, 2);
switch (result) {
    case Result.Success<Integer> s -> System.out.println("Result: " + s.value());
    case Result.Failure<Integer> f -> System.out.println("Error: " + f.error());
}

static Result<Integer> divide(int a, int b) {
    if (b == 0) return Result.failure("Division by zero");
    return Result.success(a / b);
}
```

---

## 9. Exercises

### Exercise 1 – Generic Triple
Create a `Triple<A, B, C>` class similar to `Pair`. Add a static factory method and a `toList()` method that returns `List<Object>`.

### Exercise 2 – Generic Min/Max
Write a generic method `<T extends Comparable<T>> T clamp(T value, T min, T max)` that returns `min` if `value < min`, `max` if `value > max`, or `value` otherwise.

### Exercise 3 – Generic Filter
Write a generic method `<T> List<T> filter(List<T> list, Predicate<T> predicate)` without using the Stream API.

### Exercise 4 – Typed Cache
Implement a `TypedCache` that stores `Object` values mapped by `String` keys but retrieves them cast to the expected type:
```java
TypedCache cache = new TypedCache();
cache.put("name", "Alice");
cache.put("age", 30);

String name = cache.get("name", String.class);    // "Alice"
Integer age  = cache.get("age",  Integer.class);  // 30
```

### Exercise 5 – Wildcard Usage
Given these types:
```java
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}
```
Write:
- `void feed(List<? extends Animal> animals)` that iterates and prints each animal
- `void addDogs(List<? super Dog> list)` that adds 3 new `Dog` instances

Demonstrate calling them with different list types.

---

> ⬅️ Previous: [Module 07 – Collections Framework](07-collections-framework.md) | ➡️ Next: [Module 09 – Exception Handling](09-exception-handling.md)
