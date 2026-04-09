# Module 01 – Introduction to Java & Environment Setup

## Table of Contents
1. [What is Java?](#1-what-is-java)
2. [JVM, JRE, and JDK](#2-jvm-jre-and-jdk)
3. [Installing Java 23](#3-installing-java-23)
4. [Your First Program – Hello, World!](#4-your-first-program--hello-world)
5. [Compiling and Running Java Code](#5-compiling-and-running-java-code)
6. [IDE Options](#6-ide-options)
7. [Exercises](#7-exercises)

---

## 1. What is Java?

Java is a **general-purpose, object-oriented, statically typed** programming language released by Sun Microsystems in 1995 (now maintained by Oracle). Key characteristics:

| Feature | Description |
|---------|-------------|
| **Write Once, Run Anywhere** | Compiled to bytecode that runs on any JVM |
| **Strongly typed** | Types are checked at compile time |
| **Automatic memory management** | Garbage Collector handles object lifecycle |
| **Multi-paradigm** | OOP + functional features (since Java 8) |
| **Huge ecosystem** | Largest open-source library ecosystem |

Java 23 is the latest release (September 2024) and includes exciting features like **unnamed classes**, **implicitly declared main**, and further **pattern matching** improvements.

---

## 2. JVM, JRE, and JDK

```
┌──────────────────────────────────────┐
│               JDK                    │
│  ┌────────────────────────────────┐  │
│  │              JRE               │  │
│  │  ┌──────────────────────────┐  │  │
│  │  │          JVM             │  │  │
│  │  └──────────────────────────┘  │  │
│  │  Java Class Libraries          │  │
│  └────────────────────────────────┘  │
│  Compiler (javac), Debugger, Tools   │
└──────────────────────────────────────┘
```

| Component | Full Name | Purpose |
|-----------|-----------|---------|
| **JVM** | Java Virtual Machine | Executes bytecode (`.class` files) |
| **JRE** | Java Runtime Environment | JVM + standard class libraries (needed to *run* Java) |
| **JDK** | Java Development Kit | JRE + compiler + development tools (needed to *write & compile* Java) |

---

## 3. Installing Java 23

### Option A – SDKMAN! (Recommended for Linux/macOS)

```bash
# Install SDKMAN!
curl -s "https://get.sdkman.io" | bash

# Open a new terminal, then install Java 23
sdk install java 23-tem

# Verify installation
java --version
```

### Option B – Direct download

1. Go to <https://adoptium.net/> (Eclipse Temurin) or <https://jdk.java.net/23/> (OpenJDK)
2. Download the installer for your OS
3. Follow the installer steps
4. Verify:

```bash
java --version
# Expected output similar to:
# openjdk 23 2024-09-17
# OpenJDK Runtime Environment Temurin-23+37 (build 23+37)
# OpenJDK 64-Bit Server VM Temurin-23+37 (build 23+37, mixed mode)

javac --version
# javac 23
```

### Setting JAVA_HOME (optional but common)

```bash
# macOS / Linux – add to ~/.bashrc or ~/.zshrc
export JAVA_HOME=$HOME/.sdkman/candidates/java/current
export PATH=$JAVA_HOME/bin:$PATH
```

---

## 4. Your First Program – Hello, World!

Create a file named `HelloWorld.java`:

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### Java 23 Shortcut – Unnamed Classes (Preview Feature)

Java 23 introduces **unnamed classes** (JEP 463), letting you write simple programs without a class declaration:

```java
// Save as Hello.java
void main() {
    System.out.println("Hello, World!");
}
```

Run with preview features enabled:

```bash
java --enable-preview --source 23 Hello.java
```

---

## 5. Compiling and Running Java Code

### Traditional flow

```bash
# 1. Compile source file to bytecode
javac HelloWorld.java
# Produces: HelloWorld.class

# 2. Run the class
java HelloWorld
# Output: Hello, World!
```

### Single-file source code launcher (Java 11+)

```bash
# No explicit compilation needed for single-file programs
java HelloWorld.java
```

### Anatomy of a Java program

```java
// 1. Package declaration (optional)
package com.example;

// 2. Import statements
import java.util.List;

// 3. Class declaration (filename must match class name)
public class HelloWorld {

    // 4. main method – entry point
    public static void main(String[] args) {
        // 5. Statements
        System.out.println("Hello, World!");
    }
}
```

| Element | Explanation |
|---------|-------------|
| `public` | Access modifier – visible from anywhere |
| `class HelloWorld` | Declares a class named `HelloWorld` |
| `static` | Method belongs to the class, not an instance |
| `void` | Method returns nothing |
| `main` | Special method name JVM looks for as entry point |
| `String[] args` | Command-line arguments |

---

## 6. IDE Options

| IDE | Best For | Download |
|-----|---------|----------|
| **IntelliJ IDEA Community** | Professional Java development | <https://www.jetbrains.com/idea/> |
| **VS Code + Extension Pack for Java** | Lightweight, polyglot developers | <https://code.visualstudio.com/> |
| **Eclipse IDE** | Enterprise/legacy projects | <https://www.eclipse.org/> |
| **NetBeans** | Beginners, full Java EE support | <https://netbeans.apache.org/> |

**Recommended**: IntelliJ IDEA Community Edition (free) provides the best Java experience with smart completions, refactoring, and built-in Java 23 support.

---

## 7. Exercises

### Exercise 1 – Setup Verification
1. Install Java 23 using your preferred method.
2. Open a terminal and run `java --version`. Paste the output.
3. Run `javac --version`. Confirm it shows version 23.

### Exercise 2 – Hello, World! Variations
Write and run the following programs:

a) Print your name:
```
Hello, <Your Name>!
```

b) Print multiple lines using separate `System.out.println` calls:
```
Java is...
...awesome!
```

c) Print using `System.out.print` (no newline) and `System.out.println` (with newline). What's the difference?

### Exercise 3 – Command-Line Arguments
Modify `HelloWorld.java` so that if a name is provided as a command-line argument, it greets that name:

```bash
java HelloWorld Alice
# Output: Hello, Alice!

java HelloWorld
# Output: Hello, World!
```

**Hint**: `args[0]` gives the first argument; use `args.length` to check how many were provided.

### Exercise 4 – Unnamed Class (Java 23 Preview)
Rewrite your HelloWorld using the unnamed class syntax. Run it with:
```bash
java --enable-preview --source 23 Hello.java
```

---

> ➡️ Next: [Module 02 – Variables, Data Types & Operators](02-variables-datatypes-operators.md)
