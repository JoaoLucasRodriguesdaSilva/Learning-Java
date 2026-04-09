# Module 11 – Java I/O & NIO

## Table of Contents
1. [I/O Streams Overview](#1-io-streams-overview)
2. [Reading and Writing Files](#2-reading-and-writing-files)
3. [Buffered Streams](#3-buffered-streams)
4. [Character Streams](#4-character-streams)
5. [Serialization](#5-serialization)
6. [NIO.2 – Path API (Java 7+)](#6-nio2--path-api-java-7)
7. [Files Utility Class](#7-files-utility-class)
8. [File Watching](#8-file-watching)
9. [Exercises](#9-exercises)

---

## 1. I/O Streams Overview

```
java.io
├── InputStream  (byte-based input)
│   ├── FileInputStream
│   ├── BufferedInputStream
│   ├── DataInputStream
│   └── ObjectInputStream
├── OutputStream (byte-based output)
│   ├── FileOutputStream
│   ├── BufferedOutputStream
│   ├── DataOutputStream
│   └── ObjectOutputStream
├── Reader       (character-based input)
│   ├── FileReader
│   ├── BufferedReader
│   └── StringReader
└── Writer       (character-based output)
    ├── FileWriter
    ├── BufferedWriter
    ├── StringWriter
    └── PrintWriter
```

**Rule of thumb**: Use **byte streams** for binary data (images, audio). Use **character streams** for text data (they handle character encoding).

---

## 2. Reading and Writing Files

### Writing text with FileWriter
```java
import java.io.*;

// Writing
try (FileWriter writer = new FileWriter("output.txt")) {
    writer.write("Hello, File!\n");
    writer.write("Second line.\n");
} catch (IOException e) {
    e.printStackTrace();
}

// Appending (second arg = true)
try (FileWriter writer = new FileWriter("output.txt", true)) {
    writer.write("Appended line.\n");
} catch (IOException e) {
    e.printStackTrace();
}
```

### Reading with FileReader
```java
try (FileReader reader = new FileReader("output.txt")) {
    int ch;
    while ((ch = reader.read()) != -1) {
        System.out.print((char) ch);
    }
} catch (FileNotFoundException e) {
    System.err.println("File not found: " + e.getMessage());
} catch (IOException e) {
    e.printStackTrace();
}
```

### Binary data with streams
```java
// Writing bytes
byte[] data = {72, 101, 108, 108, 111}; // "Hello" in ASCII

try (FileOutputStream fos = new FileOutputStream("binary.dat")) {
    fos.write(data);
} catch (IOException e) {
    e.printStackTrace();
}

// Reading bytes
try (FileInputStream fis = new FileInputStream("binary.dat")) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = fis.read(buffer)) != -1) {
        System.out.write(buffer, 0, bytesRead);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

---

## 3. Buffered Streams

**Buffering** reduces I/O operations by collecting data before writing, dramatically improving performance:

```java
// Without buffering – slow for large files
try (FileOutputStream fos = new FileOutputStream("large.txt")) {
    for (int i = 0; i < 100_000; i++) {
        fos.write(("Line " + i + "\n").getBytes()); // one I/O per line
    }
}

// With buffering – much faster
try (BufferedOutputStream bos =
        new BufferedOutputStream(new FileOutputStream("large.txt"))) {
    for (int i = 0; i < 100_000; i++) {
        bos.write(("Line " + i + "\n").getBytes()); // buffered in memory first
    }
} // buffer flushed on close
```

### BufferedReader – read line by line
```java
try (BufferedReader reader =
        new BufferedReader(new FileReader("large.txt"))) {
    String line;
    int lineCount = 0;
    while ((line = reader.readLine()) != null) {
        lineCount++;
        if (lineCount <= 5) System.out.println(line);
    }
    System.out.println("Total lines: " + lineCount);
} catch (IOException e) {
    e.printStackTrace();
}
```

### PrintWriter – convenient formatted writing
```java
try (PrintWriter pw = new PrintWriter(
        new BufferedWriter(new FileWriter("data.csv")))) {
    pw.println("name,age,city");
    pw.printf("Alice,%d,%s%n", 30, "São Paulo");
    pw.printf("Bob,%d,%s%n",   25, "Rio de Janeiro");
    pw.flush(); // ensure everything is written (auto-called on close)
} catch (IOException e) {
    e.printStackTrace();
}
```

---

## 4. Character Streams

Character streams automatically handle character encoding (charset conversion):

```java
import java.io.*;
import java.nio.charset.StandardCharsets;

// Write with specific encoding
try (OutputStreamWriter writer = new OutputStreamWriter(
        new FileOutputStream("utf8.txt"), StandardCharsets.UTF_8)) {
    writer.write("Olá, Mundo! 🌍\n");
    writer.write("こんにちは\n");
} catch (IOException e) {
    e.printStackTrace();
}

// Read with specific encoding
try (BufferedReader reader = new BufferedReader(
        new InputStreamReader(new FileInputStream("utf8.txt"),
        StandardCharsets.UTF_8))) {
    reader.lines().forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

### StringReader / StringWriter – in-memory
```java
// Write to string
StringWriter sw = new StringWriter();
PrintWriter pw = new PrintWriter(sw);
pw.println("Line 1");
pw.println("Line 2");
String content = sw.toString();
System.out.println(content);

// Read from string
try (BufferedReader br = new BufferedReader(new StringReader(content))) {
    br.lines().forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

---

## 5. Serialization

**Serialization** converts an object to a byte stream (for saving or sending). The class must implement `Serializable`:

```java
import java.io.*;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;  // important for versioning
    private String name;
    private int age;
    private transient String password;  // transient = NOT serialized

    public User(String name, int age, String password) {
        this.name     = name;
        this.age      = age;
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{name=" + name + ", age=" + age + ", password=" + password + "}";
    }
}
```

```java
User user = new User("Alice", 30, "secret123");
System.out.println(user); // User{name=Alice, age=30, password=secret123}

// Serialize to file
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("user.ser"))) {
    oos.writeObject(user);
    System.out.println("Serialized.");
}

// Deserialize from file
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("user.ser"))) {
    User loaded = (User) ois.readObject();
    System.out.println(loaded); // User{name=Alice, age=30, password=null}
    // password is null because it was transient
}
```

> **Modern alternative**: Prefer JSON (Jackson, Gson) or Protocol Buffers over Java serialization for cross-system compatibility and security.

---

## 6. NIO.2 – Path API (Java 7+)

The `java.nio.file` package provides a modern, richer API for file operations:

```java
import java.nio.file.*;

// Creating Path objects
Path path1 = Path.of("documents/report.txt");      // Java 11+
Path path2 = Paths.get("documents", "report.txt"); // Java 7+

// Path operations
System.out.println(path1.getFileName());   // report.txt
System.out.println(path1.getParent());     // documents
System.out.println(path1.getRoot());       // null (relative path)

Path absolute = path1.toAbsolutePath();
System.out.println(absolute);

// Normalizing (resolving ..)
Path messy = Path.of("a/b/../c/./d");
System.out.println(messy.normalize()); // a/c/d

// Resolving (building paths)
Path base = Path.of("/home/user");
Path resolved = base.resolve("documents/file.txt");
System.out.println(resolved); // /home/user/documents/file.txt

// Relativizing
Path rel = base.relativize(resolved);
System.out.println(rel); // documents/file.txt

// Checking
System.out.println(Files.exists(path1));
System.out.println(Files.isDirectory(path1));
System.out.println(Files.isReadable(path1));
```

---

## 7. Files Utility Class

`java.nio.file.Files` provides static methods for common file operations:

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.util.List;

Path file = Path.of("sample.txt");

// Write all text at once (creates or overwrites)
Files.writeString(file, "Hello, NIO!\nSecond line.\n");

// Append
Files.writeString(file, "Third line.\n", StandardOpenOption.APPEND);

// Write lines
List<String> lines = List.of("Line A", "Line B", "Line C");
Files.write(file, lines, StandardCharsets.UTF_8);

// Read all text
String content = Files.readString(file);
System.out.println(content);

// Read all lines
List<String> readLines = Files.readAllLines(file);
readLines.forEach(System.out::println);

// Read as stream of lines (lazy)
try (var lineStream = Files.lines(file)) {
    lineStream.filter(l -> l.contains("B"))
              .forEach(System.out::println);
}

// File management
Path dir = Path.of("mydir");
Files.createDirectory(dir);
Files.createDirectories(Path.of("a/b/c")); // creates entire hierarchy

Path copy = Path.of("copy.txt");
Files.copy(file, copy, StandardCopyOption.REPLACE_EXISTING);
Files.move(copy, Path.of("moved.txt"), StandardCopyOption.REPLACE_EXISTING);
Files.delete(Path.of("moved.txt"));
// Files.deleteIfExists(path); // no exception if not found

// File metadata
System.out.println(Files.size(file));
System.out.println(Files.getLastModifiedTime(file));

// Walking directory tree
try (var stream = Files.walk(Path.of("."))) {
    stream.filter(Files::isRegularFile)
          .filter(p -> p.toString().endsWith(".java"))
          .forEach(System.out::println);
}

// Listing directory contents
try (var stream = Files.list(dir)) {
    stream.forEach(System.out::println);
}
```

### Reading/Writing with specific encodings (NIO.2)
```java
// Write bytes
byte[] bytes = "Binary data".getBytes(StandardCharsets.UTF_8);
Files.write(Path.of("bin.dat"), bytes);

// Read bytes
byte[] read = Files.readAllBytes(Path.of("bin.dat"));
System.out.println(new String(read, StandardCharsets.UTF_8));
```

---

## 8. File Watching

`WatchService` notifies your program of file system changes:

```java
import java.nio.file.*;

Path dir = Path.of(".");

try (WatchService watcher = FileSystems.getDefault().newWatchService()) {
    dir.register(watcher,
        StandardWatchEventKinds.ENTRY_CREATE,
        StandardWatchEventKinds.ENTRY_DELETE,
        StandardWatchEventKinds.ENTRY_MODIFY);

    System.out.println("Watching directory: " + dir.toAbsolutePath());
    System.out.println("Press Ctrl+C to stop.");

    while (true) {
        WatchKey key = watcher.take(); // blocks until event
        for (WatchEvent<?> event : key.pollEvents()) {
            WatchEvent.Kind<?> kind = event.kind();
            if (kind == StandardWatchEventKinds.OVERFLOW) continue;

            @SuppressWarnings("unchecked")
            WatchEvent<Path> pathEvent = (WatchEvent<Path>) event;
            Path changed = dir.resolve(pathEvent.context());

            System.out.println(kind.name() + ": " + changed);
        }
        key.reset(); // re-register to receive future events
    }
}
```

---

## 9. Exercises

### Exercise 1 – Word Counter
Write a program that reads a text file and counts:
- Total number of lines
- Total number of words
- Total number of characters (excluding newlines)
- The 5 most frequent words

### Exercise 2 – CSV Parser
Write a simple CSV parser that reads a file with this format:
```
name,age,city
Alice,30,São Paulo
Bob,25,Rio de Janeiro
```
Parse each row into a `Person` record and print a formatted table.

### Exercise 3 – File Copy with Progress
Write a method `copyFile(Path source, Path target)` that:
- Copies a file in chunks of 8 KB
- Prints progress: `"Copied X of Y bytes (Z%)"`

### Exercise 4 – Directory Statistics
Write a program that accepts a directory path and prints:
- Total number of files
- Total size in bytes
- Breakdown by file extension (e.g., `.txt: 5 files, 12.3 KB`)

### Exercise 5 – Log File Analyzer
Given a log file where each line starts with `[LEVEL] timestamp - message`, write a program that:
- Groups log entries by level (INFO, WARN, ERROR)
- Finds all ERROR entries
- Counts occurrences by level
- Writes a summary to a new file

### Exercise 6 – Simple Configuration Reader
Create a `Config` class that reads a `.properties` file (key=value pairs) and:
- Provides `getString(key)`, `getInt(key)`, `getBoolean(key)` methods
- Returns a default value if the key is missing
- Saves modified config back to the file

---

> ⬅️ Previous: [Module 10 – Functional Programming](10-functional-programming.md) | ➡️ Next: [Module 12 – Concurrency & Virtual Threads](12-concurrency-and-virtual-threads.md)
