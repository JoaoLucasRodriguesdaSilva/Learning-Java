# Module 14 – Advanced Topics

## Table of Contents
1. [JVM Internals & Memory Model](#1-jvm-internals--memory-model)
2. [Garbage Collection](#2-garbage-collection)
3. [Reflection API](#3-reflection-api)
4. [Annotations](#4-annotations)
5. [Java Module System (JPMS)](#5-java-module-system-jpms)
6. [Design Patterns in Java](#6-design-patterns-in-java)
7. [Performance Profiling Tips](#7-performance-profiling-tips)
8. [Exercises](#8-exercises)

---

## 1. JVM Internals & Memory Model

### JVM Memory Areas

```
JVM Memory
├── Heap (garbage collected)
│   ├── Young Generation
│   │   ├── Eden Space
│   │   └── Survivor Spaces (S0, S1)
│   └── Old Generation (Tenured)
├── Non-Heap
│   ├── Metaspace (class metadata, unlimited by default)
│   └── Code Cache (JIT-compiled code)
├── Stack (per thread)
│   └── Stack Frames (local vars, operand stack, frame data)
├── PC Registers (per thread)
└── Native Method Stacks (per thread)
```

### Understanding the stack vs heap

```java
public class MemoryExample {
    public static void main(String[] args) {
        int x = 5;             // x stored on stack
        String s = "hello";    // reference on stack, object in string pool (heap)
        Object obj = new Object(); // reference on stack, object on heap
    }

    // Each method call gets its own stack frame
    static int factorial(int n) {         // n is on the stack frame
        if (n <= 1) return 1;
        return n * factorial(n - 1);      // new frame pushed for each call
    }
}
```

### JVM Flags for memory tuning
```bash
# Heap size
java -Xms512m -Xmx2g MyApp      # initial 512MB, max 2GB heap

# Metaspace
java -XX:MaxMetaspaceSize=256m MyApp

# GC logging
java -Xlog:gc MyApp

# Print GC details
java -verbose:gc MyApp
```

### Java Memory Model (JMM) – key concepts
- **Visibility**: Changes made by one thread may not be visible to others without synchronization
- **Happens-before**: A guarantee that operations in one thread are visible to another
- **volatile**: Ensures visibility (but not atomicity)
- **synchronized**: Ensures both visibility and atomicity

```java
// Without volatile – second thread may never see the change
class Flag {
    boolean running = true;  // NOT volatile

    void stop() { running = false; }          // Thread A
    void work() { while (running) { /* ... */ } } // Thread B – may loop forever!
}

// With volatile
class SafeFlag {
    volatile boolean running = true;  // change is visible to all threads immediately
}
```

---

## 2. Garbage Collection

The JVM automatically manages memory by collecting objects that are no longer reachable.

### GC Algorithms

| GC | Flag | Best For |
|----|------|---------|
| **G1GC** (default Java 9+) | `-XX:+UseG1GC` | Balanced throughput + latency |
| **ZGC** (default Java 23 generational) | `-XX:+UseZGC` | Ultra-low latency (<1ms pauses) |
| **Shenandoah** | `-XX:+UseShenandoahGC` | Very low pause times |
| **Parallel GC** | `-XX:+UseParallelGC` | Maximum throughput |
| **Serial GC** | `-XX:+UseSerialGC` | Single-threaded, small heaps |

### Object lifecycle
```java
// Object is eligible for GC when no strong references remain
StringBuilder sb = new StringBuilder("Hello");
sb = null;   // original object is now eligible for GC

// WeakReference – collected when memory is low
import java.lang.ref.*;

WeakReference<byte[]> weakRef = new WeakReference<>(new byte[1024 * 1024]);
// ... later ...
if (weakRef.get() == null) {
    System.out.println("Object was collected.");
}

// SoftReference – collected only when memory is needed (good for caches)
SoftReference<byte[]> softRef = new SoftReference<>(new byte[1024 * 1024]);

// PhantomReference – for post-finalization cleanup
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(new Object(), queue);
```

### Reducing GC pressure
```java
// Prefer StringBuilder over String concatenation in loops
// BAD – creates many intermediate String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // N allocations!
}

// GOOD – one mutable object
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);  // no extra allocations
}
String result2 = sb.toString();

// Object pooling for expensive objects
import java.util.concurrent.*;
ArrayBlockingQueue<Connection> pool = new ArrayBlockingQueue<>(10);
for (int i = 0; i < 10; i++) {
    pool.offer(createConnection());
}

Connection conn = pool.poll();
try {
    conn.execute(query);
} finally {
    pool.offer(conn);  // return to pool instead of discarding
}
```

---

## 3. Reflection API

**Reflection** allows inspecting and manipulating classes, methods, and fields at runtime:

```java
import java.lang.reflect.*;

// Getting Class object
Class<?> clazz = String.class;
Class<?> clazz2 = "hello".getClass();
Class<?> clazz3 = Class.forName("java.util.ArrayList");

// Inspecting a class
System.out.println(clazz.getName());          // java.lang.String
System.out.println(clazz.getSimpleName());    // String
System.out.println(clazz.getSuperclass());    // class java.lang.Object
System.out.println(java.util.Arrays.toString(clazz.getInterfaces())); // [Serializable, Comparable, CharSequence]

// Fields
Field[] fields = clazz.getDeclaredFields(); // all fields including private
for (Field f : fields) {
    System.out.println(f.getName() + " : " + f.getType());
}

// Methods
Method[] methods = clazz.getDeclaredMethods();
for (Method m : methods) {
    System.out.printf("%-20s | params: %d%n",
        m.getName(), m.getParameterCount());
}

// Constructors
Constructor<?>[] constructors = clazz.getDeclaredConstructors();
```

### Creating instances and invoking methods
```java
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age  = age;
    }

    private String greet() {
        return "Hi, I'm " + name + ", age " + age;
    }
}
```

```java
// Create instance via reflection
Constructor<Person> ctor = Person.class.getDeclaredConstructor(String.class, int.class);
Person person = ctor.newInstance("Alice", 30);

// Access private field
Field nameField = Person.class.getDeclaredField("name");
nameField.setAccessible(true);              // bypass private access
String name = (String) nameField.get(person);
System.out.println("Name: " + name);       // Alice

nameField.set(person, "Bob");              // modify field value

// Invoke private method
Method greetMethod = Person.class.getDeclaredMethod("greet");
greetMethod.setAccessible(true);
String greeting = (String) greetMethod.invoke(person);
System.out.println(greeting);              // Hi, I'm Bob, age 30
```

### Checking annotations at runtime
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Timed {}

// Usage
public class Service {
    @Timed
    public void process() { /* ... */ }
}

// Checking at runtime
for (Method method : Service.class.getDeclaredMethods()) {
    if (method.isAnnotationPresent(Timed.class)) {
        long start = System.nanoTime();
        method.invoke(new Service());
        long end   = System.nanoTime();
        System.out.printf("%s took %.2f ms%n",
            method.getName(), (end - start) / 1_000_000.0);
    }
}
```

> **Note**: Reflection bypasses compile-time checks and can break encapsulation. Use it sparingly. Consider it for frameworks, test utilities, and meta-programming – not regular business logic. Also note that since Java 9, module-private types may require `--add-opens` flags.

---

## 4. Annotations

Annotations provide **metadata** about code. They don't affect execution by themselves but are read by the compiler, tools, or at runtime.

### Built-in annotations
```java
@Override           // compiler verifies this overrides a superclass method
@Deprecated         // marks as outdated; triggers compiler warning on use
@SuppressWarnings   // silences specific compiler warnings
@FunctionalInterface // ensures only one abstract method

public class Example {
    @Deprecated(since = "3.0", forRemoval = true)
    public void oldMethod() { /* ... */ }

    @SuppressWarnings("unchecked")
    public void sneakyMethod() {
        List list = new ArrayList(); // raw type warning suppressed
    }
}
```

### Creating custom annotations
```java
import java.lang.annotation.*;

// Define annotation
@Retention(RetentionPolicy.RUNTIME)  // available at runtime
@Target({ElementType.METHOD, ElementType.TYPE})   // where it can be applied
public @interface Validate {
    String message() default "Validation failed";
    boolean required() default true;
    Class<?>[] groups() default {};
}

// Apply annotation
public class UserService {
    @Validate(message = "User creation failed", required = true)
    public User createUser(String name) {
        // ...
    }
}
```

### Annotation processing
```java
// Process at runtime
Method method = UserService.class.getMethod("createUser", String.class);
Validate validate = method.getAnnotation(Validate.class);
if (validate != null) {
    System.out.println("Message: " + validate.message());
    System.out.println("Required: " + validate.required());
}

// Repeatable annotations (Java 8+)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(Roles.class)
public @interface Role {
    String value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Roles {
    Role[] value();
}

// Usage
@Role("ADMIN")
@Role("MANAGER")
public void adminAction() { /* ... */ }
```

---

## 5. Java Module System (JPMS)

Java 9 introduced the **Java Platform Module System** (Project Jigsaw) to encapsulate packages and control dependencies:

### module-info.java
```java
// module-info.java – place at the root of your source tree
module com.example.myapp {
    // Dependencies
    requires java.base;           // implicit – always included
    requires java.sql;
    requires com.google.gson;

    // Transitive dependency (consumers of this module also get it)
    requires transitive java.logging;

    // Exports – packages available to other modules
    exports com.example.myapp.api;
    exports com.example.myapp.model;

    // Exports only to specific modules
    exports com.example.myapp.internal to com.example.tests;

    // Opens – allows reflection access (e.g., for frameworks)
    opens com.example.myapp.model to com.google.gson;

    // Service provider
    provides com.example.myapp.api.DataSource
        with com.example.myapp.impl.DatabaseDataSource;

    // Service consumer
    uses com.example.myapp.api.Plugin;
}
```

### Module structure
```
my-project/
├── src/
│   └── com.example.myapp/
│       ├── module-info.java
│       └── com/example/myapp/
│           ├── Main.java
│           ├── api/
│           └── impl/
└── test/
    └── com.example.tests/
        ├── module-info.java
        └── ...
```

### Compiling and running modular code
```bash
# Compile
javac -d out/com.example.myapp \
      --module-path lib \
      src/com.example.myapp/module-info.java \
      src/com.example.myapp/com/example/myapp/*.java

# Run
java --module-path out:lib \
     --module com.example.myapp/com.example.myapp.Main
```

---

## 6. Design Patterns in Java

### Creational Patterns

**Builder** – step-by-step object construction:
```java
public class HttpRequest {
    private final String url;
    private final String method;
    private final java.util.Map<String, String> headers;
    private final String body;
    private final int timeout;

    private HttpRequest(Builder b) {
        this.url     = b.url;
        this.method  = b.method;
        this.headers = java.util.Collections.unmodifiableMap(b.headers);
        this.body    = b.body;
        this.timeout = b.timeout;
    }

    public static class Builder {
        private final String url;
        private String method = "GET";
        private final java.util.Map<String, String> headers = new java.util.HashMap<>();
        private String body = null;
        private int timeout = 30;

        public Builder(String url) { this.url = url; }
        public Builder method(String m)  { this.method = m;  return this; }
        public Builder header(String k, String v) { headers.put(k, v); return this; }
        public Builder body(String b)    { this.body = b;    return this; }
        public Builder timeout(int t)    { this.timeout = t; return this; }
        public HttpRequest build()       { return new HttpRequest(this); }
    }
}
```

```java
HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token123")
    .body("{\"name\": \"Alice\"}")
    .timeout(10)
    .build();
```

**Factory Method**:
```java
public interface Notification {
    void send(String message);
}

public class NotificationFactory {
    public static Notification create(String type) {
        return switch (type) {
            case "email" -> new EmailNotification();
            case "sms"   -> new SmsNotification();
            case "push"  -> new PushNotification();
            default      -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}
```

### Structural Patterns

**Decorator** – add behavior without subclassing:
```java
public interface TextProcessor {
    String process(String text);
}

public class UpperCaseProcessor implements TextProcessor {
    public String process(String text) { return text.toUpperCase(); }
}

public class TrimProcessor implements TextProcessor {
    private final TextProcessor delegate;
    public TrimProcessor(TextProcessor d) { this.delegate = d; }
    public String process(String text) { return delegate.process(text.trim()); }
}

public class LoggingProcessor implements TextProcessor {
    private final TextProcessor delegate;
    public LoggingProcessor(TextProcessor d) { this.delegate = d; }
    public String process(String text) {
        System.out.println("Processing: " + text);
        String result = delegate.process(text);
        System.out.println("Result: " + result);
        return result;
    }
}
```

```java
TextProcessor pipeline = new LoggingProcessor(
    new TrimProcessor(
        new UpperCaseProcessor()
    )
);
pipeline.process("  hello world  "); // HELLO WORLD
```

### Behavioral Patterns

**Observer** (Event System):
```java
import java.util.*;
import java.util.function.*;

public class EventBus<T> {
    private final List<Consumer<T>> listeners = new ArrayList<>();

    public void subscribe(Consumer<T> listener) { listeners.add(listener); }
    public void unsubscribe(Consumer<T> listener) { listeners.remove(listener); }
    public void publish(T event) { listeners.forEach(l -> l.accept(event)); }
}
```

```java
EventBus<String> bus = new EventBus<>();
bus.subscribe(msg -> System.out.println("Logger: " + msg));
bus.subscribe(msg -> System.out.println("Auditor: " + msg));

bus.publish("User logged in");
// Logger: User logged in
// Auditor: User logged in
```

**Strategy**:
```java
public interface SortStrategy<T extends Comparable<T>> {
    void sort(List<T> list);
}

public class Sorter<T extends Comparable<T>> {
    private SortStrategy<T> strategy;

    public Sorter(SortStrategy<T> strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(SortStrategy<T> strategy) {
        this.strategy = strategy;
    }

    public void sort(List<T> list) { strategy.sort(list); }
}

// Usage with lambdas
Sorter<Integer> sorter = new Sorter<>(list -> Collections.sort(list));
sorter.sort(myList);

sorter.setStrategy(list -> list.sort(Comparator.reverseOrder()));
sorter.sort(myList);
```

---

## 7. Performance Profiling Tips

### Microbenchmarking with JMH
```java
// Add JMH dependency and use @Benchmark annotation
import org.openjdk.jmh.annotations.*;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(java.util.concurrent.TimeUnit.MICROSECONDS)
public class StringBenchmark {

    @Benchmark
    public String concatenation() {
        String result = "";
        for (int i = 0; i < 100; i++) {
            result += i;
        }
        return result;
    }

    @Benchmark
    public String stringBuilder() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 100; i++) {
            sb.append(i);
        }
        return sb.toString();
    }
}
```

### Common performance tips
```java
// 1. Prefer local variables – faster than field access
double localPi = Math.PI;  // cache field access in hot loops

// 2. ArrayList vs LinkedList
List<Integer> arrayList = new ArrayList<>(10_000); // random access O(1)
// LinkedList: avoid for random access (O(n))

// 3. StringBuilder for concatenation in loops
// 4. Use primitives instead of boxed types when possible
int sum = 0;                // fast
Integer boxedSum = 0;       // slower (unboxing on each operation)

// 5. Avoid unnecessary autoboxing
Map<Integer, Integer> map = new HashMap<>();
// Use Trove or Eclipse Collections for primitive maps in perf-critical code

// 6. Reuse expensive objects (DateTimeFormatter, Pattern, etc.)
// BAD – compiled on every call
Pattern.compile("\\d+").matcher(input).matches();

// GOOD – compile once
private static final Pattern DIGITS = Pattern.compile("\\d+");
DIGITS.matcher(input).matches();

// 7. Use Arrays.fill for bulk initialization
int[] arr = new int[1000];
java.util.Arrays.fill(arr, 42);  // faster than a loop

// 8. System.arraycopy for bulk copies
System.arraycopy(src, 0, dest, 0, length);  // native, fastest
```

### JVM diagnostic flags
```bash
# Print compilation activity
java -XX:+PrintCompilation MyApp

# Print GC details
java -Xlog:gc* MyApp

# Flight Recorder (low-overhead profiling)
java -XX:+FlightRecorder -XX:StartFlightRecording=duration=60s,filename=recording.jfr MyApp

# Analyze with JDK Mission Control
jmc
```

---

## 8. Exercises

### Exercise 1 – Reflection-based Mapper
Build an `ObjectMapper` class with a method `<T> T map(Map<String, Object> data, Class<T> clazz)` that:
- Creates an instance using the no-arg constructor
- Sets each field from the map using reflection
- Handles type conversion (String → int, String → boolean, etc.)

### Exercise 2 – Custom Annotation Processor
Create an annotation `@NotNull` and a `Validator` class that:
- Accepts any object
- Finds all fields annotated with `@NotNull`
- Throws `ValidationException` if any are null
- Returns a list of validation errors instead of throwing (alternative method)

### Exercise 3 – Simple IoC Container
Build a simple Inversion of Control (IoC) container:
- Supports `@Inject` annotation on constructor parameters
- `register(Class<?> impl)` to register an implementation
- `get(Class<?> type)` to retrieve and auto-wire an instance

### Exercise 4 – Event System with Generics
Enhance the `EventBus` to:
- Support typed events using generics
- Allow subscribing to specific event subtypes
- Provide `subscribeOnce` (auto-unsubscribes after first event)
- Support priorities (higher priority listeners are called first)

### Exercise 5 – Module System
Create a multi-module Maven/Gradle project with:
- `com.example.api` module: defines `UserRepository` interface
- `com.example.impl` module: implements it with in-memory storage
- `com.example.app` module: uses the API without depending on the impl

Use the module system's `provides`/`uses` service loading mechanism.

### Exercise 6 – Performance Comparison
Benchmark the following (use `System.nanoTime()`):
1. Array vs ArrayList for 1 million random-access reads
2. HashMap vs TreeMap for 1 million lookups
3. StringBuilder vs String concatenation for building a 10,000-char string
4. Stream vs for-loop for summing 1 million integers

Print the results in a formatted table.

---

## What's Next?

Congratulations on completing the Java 23 learning path! 🎉

Here are some areas to explore next:

| Area | Topics |
|------|--------|
| **Web Development** | Spring Boot, Micronaut, Quarkus |
| **Testing** | JUnit 5, Mockito, TestContainers, AssertJ |
| **Build Tools** | Maven, Gradle |
| **Persistence** | JPA/Hibernate, Spring Data, JDBC |
| **Microservices** | Spring Cloud, gRPC, REST APIs |
| **Reactive Programming** | Project Reactor, RxJava, WebFlux |
| **Native Compilation** | GraalVM Native Image |
| **Cloud** | AWS SDK for Java, Azure SDK, GCP |

---

> ⬅️ Previous: [Module 13 – Java 23 New Features](13-java23-new-features.md) | ⬆️ Back to [README](../README.md)
