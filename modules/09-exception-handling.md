# Module 09 – Exception Handling

## Table of Contents
1. [What Are Exceptions?](#1-what-are-exceptions)
2. [Exception Hierarchy](#2-exception-hierarchy)
3. [try / catch / finally](#3-try--catch--finally)
4. [Multi-catch and Rethrowing](#4-multi-catch-and-rethrowing)
5. [try-with-resources](#5-try-with-resources)
6. [Checked vs Unchecked Exceptions](#6-checked-vs-unchecked-exceptions)
7. [Custom Exceptions](#7-custom-exceptions)
8. [Best Practices](#8-best-practices)
9. [Exercises](#9-exercises)

---

## 1. What Are Exceptions?

An **exception** is an event that disrupts the normal flow of a program. Java uses exceptions (objects) to signal errors and abnormal conditions.

```java
int[] array = {1, 2, 3};
System.out.println(array[5]);  // ArrayIndexOutOfBoundsException

String s = null;
s.length();                    // NullPointerException

int result = 10 / 0;           // ArithmeticException

int n = Integer.parseInt("abc"); // NumberFormatException
```

Without handling, the JVM prints a **stack trace** and terminates the current thread.

---

## 2. Exception Hierarchy

```
java.lang.Throwable
├── Error               (serious JVM problems – do NOT catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── RuntimeException (unchecked – compiler doesn't require handling)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── IllegalArgumentException
    │   ├── IllegalStateException
    │   ├── NumberFormatException
    │   ├── ClassCastException
    │   └── ArithmeticException
    └── (Other Exceptions – checked – must be caught or declared)
        ├── IOException
        │   ├── FileNotFoundException
        │   └── EOFException
        ├── SQLException
        ├── ParseException
        └── InterruptedException
```

---

## 3. try / catch / finally

```java
try {
    // Code that might throw an exception
    int result = 10 / 0;
    System.out.println("Result: " + result);  // never reached
} catch (ArithmeticException e) {
    // Handle specific exception
    System.out.println("Error: " + e.getMessage()); // Error: / by zero
} finally {
    // ALWAYS executed, whether exception occurred or not
    System.out.println("Finally block runs.");
}
```

### Multiple catch blocks
```java
String input = "abc";
try {
    int number = Integer.parseInt(input);
    int[] arr  = new int[number];
    arr[10] = 42;
} catch (NumberFormatException e) {
    System.out.println("Invalid number format: " + input);
} catch (NegativeArraySizeException e) {
    System.out.println("Array size cannot be negative.");
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Array index out of bounds: " + e.getMessage());
} finally {
    System.out.println("Cleanup done.");
}
```

> **Order matters**: catch more specific exceptions before more general ones.

```java
try {
    // ...
} catch (Exception e) {      // Wrong! Catches everything first
    // ...
} catch (IOException e) {    // Compile error – already caught above!
    // ...
}
```

### Catching parent exception types
```java
try {
    riskyOperation();
} catch (IOException e) {       // catches FileNotFoundException, etc.
    System.err.println("I/O error: " + e.getMessage());
    e.printStackTrace();        // print full stack trace
}
```

### Exception information
```java
try {
    int[] arr = new int[5];
    arr[10] = 1;
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Message:  " + e.getMessage());   // Index 10 out of bounds for length 5
    System.out.println("Class:    " + e.getClass().getName());
    System.out.println("Cause:    " + e.getCause());
    // e.printStackTrace();  // detailed stack trace to System.err
}
```

---

## 4. Multi-catch and Rethrowing

### Multi-catch (Java 7+) – handle multiple exceptions the same way
```java
try {
    processFile("data.txt");
} catch (IOException | ParseException e) {
    System.err.println("Processing failed: " + e.getMessage());
    // e is effectively final in a multi-catch block
}
```

### Rethrowing
```java
public void readConfig(String path) throws IOException {
    try {
        // ...
    } catch (IOException e) {
        System.err.println("Failed to read config: " + path);
        throw e;  // rethrow the original exception
    }
}

// Wrapping in a different exception
public void loadData() {
    try {
        readConfig("config.json");
    } catch (IOException e) {
        throw new RuntimeException("Application startup failed", e); // e is the cause
    }
}
```

### Exception chaining
```java
catch (SQLException e) {
    throw new DataAccessException("Query failed", e);  // wraps original cause
}

// Later you can retrieve the original cause
try {
    loadData();
} catch (RuntimeException e) {
    Throwable cause = e.getCause();  // the original IOException
    System.out.println("Root cause: " + cause);
}
```

---

## 5. try-with-resources

Automatically closes resources that implement `AutoCloseable`:

```java
// Without try-with-resources (verbose, error-prone)
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try { reader.close(); } catch (IOException e) { /* ignore */ }
    }
}

// With try-with-resources (cleaner)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
// reader.close() is called automatically
```

### Multiple resources
```java
try (
    var input  = new FileInputStream("input.txt");
    var output = new FileOutputStream("output.txt")
) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = input.read(buffer)) != -1) {
        output.write(buffer, 0, bytesRead);
    }
} catch (IOException e) {
    System.err.println("Copy failed: " + e.getMessage());
}
// Both streams are closed in reverse order (output first, then input)
```

### Custom AutoCloseable
```java
public class DatabaseConnection implements AutoCloseable {
    private final String url;
    private boolean open;

    public DatabaseConnection(String url) {
        this.url  = url;
        this.open = true;
        System.out.println("Connecting to: " + url);
    }

    public void query(String sql) {
        if (!open) throw new IllegalStateException("Connection is closed");
        System.out.println("Executing: " + sql);
    }

    @Override
    public void close() {
        if (open) {
            open = false;
            System.out.println("Connection to " + url + " closed.");
        }
    }
}
```

```java
try (var conn = new DatabaseConnection("jdbc:postgresql://localhost/mydb")) {
    conn.query("SELECT * FROM users");
}
// Connecting to: jdbc:postgresql://localhost/mydb
// Executing: SELECT * FROM users
// Connection to jdbc:postgresql://localhost/mydb closed.
```

---

## 6. Checked vs Unchecked Exceptions

| | Checked | Unchecked |
|--|---------|-----------|
| **Extends** | `Exception` | `RuntimeException` |
| **Compiler requires** | Must catch or declare `throws` | No requirement |
| **Typical use** | Recoverable situations (file not found) | Programming errors (null pointer, bad argument) |
| **Examples** | `IOException`, `SQLException` | `NullPointerException`, `IllegalArgumentException` |

```java
// Checked – must be declared or caught
public String readFile(String path) throws IOException {
    return Files.readString(Path.of(path));  // may throw IOException
}

// Unchecked – no declaration needed
public int divide(int a, int b) {
    if (b == 0) throw new ArithmeticException("Division by zero");
    return a / b;
}
```

---

## 7. Custom Exceptions

### Unchecked custom exception (most common)
```java
public class InsufficientFundsException extends RuntimeException {
    private final double amount;
    private final double balance;

    public InsufficientFundsException(double amount, double balance) {
        super(String.format("Insufficient funds: tried to withdraw %.2f but balance is %.2f",
              amount, balance));
        this.amount  = amount;
        this.balance = balance;
    }

    public double getAmount()  { return amount; }
    public double getBalance() { return balance; }
}
```

### Checked custom exception
```java
public class UserNotFoundException extends Exception {
    private final int userId;

    public UserNotFoundException(int userId) {
        super("User not found with id: " + userId);
        this.userId = userId;
    }

    public UserNotFoundException(int userId, Throwable cause) {
        super("User not found with id: " + userId, cause);
        this.userId = userId;
    }

    public int getUserId() { return userId; }
}
```

```java
public class BankAccount {
    private double balance;

    public void withdraw(double amount) {
        if (amount > balance) {
            throw new InsufficientFundsException(amount, balance);
        }
        balance -= amount;
    }
}

BankAccount account = new BankAccount();
account.deposit(100.0);
try {
    account.withdraw(150.0);
} catch (InsufficientFundsException e) {
    System.out.println(e.getMessage());
    System.out.printf("Tried: %.2f, Available: %.2f%n",
                      e.getAmount(), e.getBalance());
}
```

---

## 8. Best Practices

### ✅ Do
- Catch specific exceptions (not just `Exception`)
- Provide meaningful messages
- Log the exception before/while handling it
- Clean up resources in `finally` or use try-with-resources
- Wrap low-level exceptions in higher-level ones (with cause preserved)
- Throw early, catch late

### ❌ Don't
- Swallow exceptions silently:
  ```java
  // BAD!
  catch (Exception e) { /* do nothing */ }
  ```
- Use exceptions for flow control (performance cost)
- Catch `Error` or `Throwable` (unless you have a very specific reason)
- Declare `throws Exception` generically

```java
// BAD – catching too broadly
try {
    doSomething();
} catch (Exception e) {
    // silent swallow!
}

// BETTER
try {
    doSomething();
} catch (IOException e) {
    logger.error("Failed to read data", e);
    throw new ServiceException("Data retrieval failed", e);
}
```

---

## 9. Exercises

### Exercise 1 – Safe Division
Write a method `safeDivide(int a, int b)` that returns an `Optional<Double>`:
- Returns `Optional.of(a / (double) b)` if `b != 0`
- Returns `Optional.empty()` if `b == 0`
- No exception should propagate to the caller

### Exercise 2 – Input Validation
Write a method `parseAge(String input)` that:
- Parses the input as an integer
- Throws `IllegalArgumentException` if the value is not between 0 and 150
- Throws a custom `AgeParseException` (checked) that wraps the `NumberFormatException` if input is not a number

### Exercise 3 – File Reader with Retry
Write a method `readFileWithRetry(String path, int maxRetries)` that:
- Tries to read a file using `Files.readString()`
- Retries up to `maxRetries` times on `IOException`
- Throws `IOException` after all retries are exhausted

### Exercise 4 – Custom Resource
Create a `Timer` class that implements `AutoCloseable`. When opened, it records the start time. When closed, it prints the elapsed time in milliseconds. Use it in a try-with-resources block.

### Exercise 5 – Exception Chain
Simulate a 3-layer architecture:
- `DatabaseLayer.fetchUser(int id)` – throws `SQLException` if id < 0
- `ServiceLayer.getUser(int id)` – calls database, wraps `SQLException` in `UserServiceException`
- `Controller.handleRequest(int id)` – calls service, catches `UserServiceException` and prints user-friendly message

Demonstrate the full exception chain with `getCause()`.

---

> ⬅️ Previous: [Module 08 – Generics](08-generics.md) | ➡️ Next: [Module 10 – Functional Programming](10-functional-programming.md)
