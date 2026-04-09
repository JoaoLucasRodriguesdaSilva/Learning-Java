# Module 12 – Concurrency & Virtual Threads

## Table of Contents
1. [Thread Basics](#1-thread-basics)
2. [Runnable and Callable](#2-runnable-and-callable)
3. [Thread Safety & Synchronization](#3-thread-safety--synchronization)
4. [ExecutorService & Thread Pools](#4-executorservice--thread-pools)
5. [Future and CompletableFuture](#5-future-and-completablefuture)
6. [Concurrent Collections](#6-concurrent-collections)
7. [Virtual Threads (Java 21+)](#7-virtual-threads-java-21)
8. [Structured Concurrency (Java 21+)](#8-structured-concurrency-java-21)
9. [Exercises](#9-exercises)

---

## 1. Thread Basics

Java supports **multithreading** – running multiple threads simultaneously within one process.

```java
// Method 1: Extend Thread
public class MyThread extends Thread {
    private final String task;

    public MyThread(String task) {
        this.task = task;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(getName() + " – " + task + " #" + i);
            try {
                Thread.sleep(100); // pause 100ms
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); // restore flag
                return;
            }
        }
    }
}
```

```java
Thread t1 = new MyThread("Task A");
Thread t2 = new MyThread("Task B");

t1.start();   // start() creates a new thread; DO NOT call run() directly
t2.start();

t1.join();    // wait for t1 to finish
t2.join();    // wait for t2 to finish

System.out.println("Both threads done.");
```

### Thread states
```
NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED
```

### Thread properties
```java
Thread t = Thread.currentThread();
System.out.println(t.getName());         // Thread-0
System.out.println(t.getId());           // long ID
System.out.println(t.getPriority());     // 5 (NORM_PRIORITY)
System.out.println(t.isDaemon());        // false
System.out.println(t.getState());        // RUNNABLE

t.setName("worker-1");
t.setPriority(Thread.MAX_PRIORITY);      // 10
t.setDaemon(true);                       // daemon threads don't prevent JVM exit
```

---

## 2. Runnable and Callable

### Runnable – no return value
```java
Runnable task = () -> {
    System.out.println("Running on: " + Thread.currentThread().getName());
};

Thread thread = new Thread(task, "my-thread");
thread.start();
```

### Callable – returns a value (or throws)
```java
import java.util.concurrent.*;

Callable<Integer> compute = () -> {
    Thread.sleep(500);
    return 42;
};

FutureTask<Integer> future = new FutureTask<>(compute);
new Thread(future).start();

Integer result = future.get(); // blocks until done
System.out.println("Result: " + result); // 42
```

---

## 3. Thread Safety & Synchronization

When multiple threads access shared data, **race conditions** can occur:

```java
// UNSAFE – race condition on counter
public class Counter {
    private int count = 0;

    public void increment() {
        count++;   // NOT atomic: read → add → write (3 separate operations)
    }

    public int getCount() { return count; }
}
```

### `synchronized` keyword
```java
public class SafeCounter {
    private int count = 0;

    // synchronized method – only one thread at a time
    public synchronized void increment() {
        count++;
    }

    // synchronized block – finer control
    public void incrementBlock() {
        synchronized (this) {
            count++;
        }
    }

    public synchronized int getCount() { return count; }
}
```

### `volatile` keyword
Ensures visibility of a variable's latest value across threads (but not atomicity):

```java
public class StopFlag {
    private volatile boolean running = true;   // visible to all threads

    public void stop() { running = false; }

    public void run() {
        while (running) {
            // do work
        }
        System.out.println("Stopped.");
    }
}
```

### Atomic classes (`java.util.concurrent.atomic`)
```java
import java.util.concurrent.atomic.*;

AtomicInteger atomicCount = new AtomicInteger(0);
atomicCount.incrementAndGet();             // thread-safe increment
atomicCount.addAndGet(5);                 // thread-safe add
atomicCount.compareAndSet(6, 0);          // CAS – set to 0 if current is 6

AtomicReference<String> atomicStr = new AtomicReference<>("initial");
atomicStr.set("updated");

AtomicLong atomicLong = new AtomicLong();
AtomicBoolean atomicBool = new AtomicBoolean(false);
```

### `ReentrantLock`
```java
import java.util.concurrent.locks.*;

public class BankAccount {
    private double balance;
    private final Lock lock = new ReentrantLock();

    public void deposit(double amount) {
        lock.lock();
        try {
            balance += amount;
        } finally {
            lock.unlock(); // always unlock in finally!
        }
    }

    public boolean withdraw(double amount) {
        lock.lock();
        try {
            if (balance >= amount) {
                balance -= amount;
                return true;
            }
            return false;
        } finally {
            lock.unlock();
        }
    }
}
```

---

## 4. ExecutorService & Thread Pools

Creating threads manually is expensive. **Thread pools** reuse threads:

```java
import java.util.concurrent.*;

// Fixed thread pool
ExecutorService executor = Executors.newFixedThreadPool(4);

for (int i = 0; i < 10; i++) {
    final int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " on " + Thread.currentThread().getName());
        Thread.sleep(100);
        return taskId * 2;  // return value (Callable)
    });
}

// Shutdown – stops accepting new tasks, waits for running ones
executor.shutdown();
executor.awaitTermination(5, TimeUnit.SECONDS);
```

### Types of executors
```java
// Single thread – sequential execution
ExecutorService single = Executors.newSingleThreadExecutor();

// Cached pool – creates threads as needed, reuses idle threads
ExecutorService cached = Executors.newCachedThreadPool();

// Scheduled pool – for periodic tasks
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

scheduler.scheduleAtFixedRate(
    () -> System.out.println("Tick: " + System.currentTimeMillis()),
    0,    // initial delay
    1,    // period
    TimeUnit.SECONDS
);

// Modern: virtual thread executor (Java 21+)
ExecutorService virtualExec = Executors.newVirtualThreadPerTaskExecutor();
```

### Collecting results with Future
```java
List<Future<Integer>> futures = new ArrayList<>();
ExecutorService exec = Executors.newFixedThreadPool(3);

for (int i = 1; i <= 5; i++) {
    final int n = i;
    futures.add(exec.submit(() -> n * n));
}

exec.shutdown();

for (Future<Integer> future : futures) {
    try {
        System.out.println("Result: " + future.get()); // blocks if not ready
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

---

## 5. Future and CompletableFuture

`CompletableFuture` provides a rich API for asynchronous programming:

```java
import java.util.concurrent.*;

// Simple async computation
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    // runs in ForkJoinPool.commonPool()
    Thread.sleep(500);
    return 42;
});

// Non-blocking – chain operations
future.thenApply(n -> n * 2)                   // transform result
      .thenApply(n -> "Result: " + n)
      .thenAccept(System.out::println)          // consume result
      .exceptionally(e -> {                     // handle exception
          System.err.println("Error: " + e.getMessage());
          return null;
      });

// Combine two futures
CompletableFuture<String> user = CompletableFuture.supplyAsync(() -> "Alice");
CompletableFuture<Integer> age  = CompletableFuture.supplyAsync(() -> 30);

CompletableFuture<String> combined = user.thenCombine(age,
    (name, a) -> name + " is " + a + " years old");
System.out.println(combined.get()); // Alice is 30 years old

// Wait for any of N futures
CompletableFuture<String> fast  = CompletableFuture.supplyAsync(() -> { Thread.sleep(100); return "fast";  });
CompletableFuture<String> slow  = CompletableFuture.supplyAsync(() -> { Thread.sleep(500); return "slow";  });
CompletableFuture<Object> first = CompletableFuture.anyOf(fast, slow);
System.out.println(first.get()); // fast

// Wait for all
CompletableFuture<Void> all = CompletableFuture.allOf(fast, slow);
all.get(); // waits for both

// Error handling
CompletableFuture.supplyAsync(() -> {
    if (Math.random() < 0.5) throw new RuntimeException("Random failure");
    return "success";
})
.handle((result, ex) -> {
    if (ex != null) return "Recovered from: " + ex.getMessage();
    return result;
})
.thenAccept(System.out::println);
```

---

## 6. Concurrent Collections

Thread-safe collection alternatives:

| Thread-safe | Replaces |
|-------------|---------|
| `ConcurrentHashMap` | `HashMap` |
| `CopyOnWriteArrayList` | `ArrayList` |
| `CopyOnWriteArraySet` | `HashSet` |
| `ConcurrentLinkedQueue` | `LinkedList` (queue use) |
| `BlockingQueue` impls | Producer-consumer pattern |

```java
import java.util.concurrent.*;

// ConcurrentHashMap – lock-stripe based (highly concurrent)
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
map.merge("a", 1, Integer::sum);   // atomic update

// CopyOnWriteArrayList – great for read-heavy, rare writes
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("a");
cowList.add("b");
// Can iterate safely even if modified concurrently

// BlockingQueue – producer-consumer
BlockingQueue<String> queue = new LinkedBlockingQueue<>(100); // capacity 100

// Producer
Thread producer = new Thread(() -> {
    try {
        for (int i = 0; i < 10; i++) {
            queue.put("item-" + i); // blocks if queue is full
            Thread.sleep(50);
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

// Consumer
Thread consumer = new Thread(() -> {
    try {
        while (true) {
            String item = queue.poll(1, TimeUnit.SECONDS); // blocks up to 1s
            if (item == null) break; // timeout
            System.out.println("Consumed: " + item);
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

producer.start();
consumer.start();
producer.join();
consumer.join();
```

---

## 7. Virtual Threads (Java 21+)

**Virtual threads** (Project Loom, finalized in Java 21) are lightweight threads managed by the JVM rather than the OS, enabling millions of concurrent threads:

```java
import java.util.concurrent.*;

// Creating a virtual thread
Thread vThread = Thread.ofVirtual().start(() ->
    System.out.println("Virtual thread: " + Thread.currentThread())
);
vThread.join();

// Named virtual thread
Thread named = Thread.ofVirtual()
    .name("my-virtual-thread")
    .start(() -> System.out.println("Running on " + Thread.currentThread().getName()));

// Virtual thread factory
ThreadFactory factory = Thread.ofVirtual().factory();
Thread t = factory.newThread(() -> System.out.println("Via factory"));
t.start();

// Using ExecutorService with virtual threads
try (ExecutorService exec = Executors.newVirtualThreadPerTaskExecutor()) {
    // Each task gets its own virtual thread
    for (int i = 0; i < 1_000_000; i++) {   // 1 million tasks!
        final int taskId = i;
        exec.submit(() -> {
            Thread.sleep(1000);              // non-blocking for virtual threads
            return taskId;
        });
    }
} // executor shutdown and awaits completion
```

### Platform vs Virtual Threads
| Feature | Platform Thread | Virtual Thread |
|---------|----------------|---------------|
| Maps to | OS thread | JVM-managed |
| Creation cost | High | Very low |
| Practical limit | ~thousands | ~millions |
| Stack size | ~1 MB | Grows as needed |
| Best for | CPU-bound tasks | I/O-bound tasks |

```java
// Measuring throughput difference
// Platform thread HTTP simulation
long startPlatform = System.currentTimeMillis();
try (var exec = Executors.newFixedThreadPool(200)) {
    for (int i = 0; i < 10_000; i++) {
        exec.submit(() -> Thread.sleep(10)); // simulate I/O
    }
}
System.out.println("Platform: " + (System.currentTimeMillis() - startPlatform) + "ms");

// Virtual thread HTTP simulation
long startVirtual = System.currentTimeMillis();
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        exec.submit(() -> Thread.sleep(10)); // non-blocking for virtual threads
    }
}
System.out.println("Virtual: " + (System.currentTimeMillis() - startVirtual) + "ms");
```

---

## 8. Structured Concurrency (Java 21+)

**Structured concurrency** (JEP 428, incubator; JEP 453, preview in Java 21/23) treats groups of related concurrent tasks as a single unit:

```java
import java.util.concurrent.*;

// Classic approach – hard to manage failures
void fetchUserData(int userId) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        Future<String> user    = scope.fork(() -> fetchUser(userId));
        Future<List<String>> orders = scope.fork(() -> fetchOrders(userId));
        Future<String> prefs   = scope.fork(() -> fetchPreferences(userId));

        scope.join();           // wait for all to complete
        scope.throwIfFailed();  // propagate exception if any task failed

        // If we reach here, all tasks succeeded
        System.out.println("User:   " + user.resultNow());
        System.out.println("Orders: " + orders.resultNow());
        System.out.println("Prefs:  " + prefs.resultNow());
    }
}

// ShutdownOnSuccess – return first successful result
void raceCondition() throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
        scope.fork(() -> { Thread.sleep(500); return "slow-server"; });
        scope.fork(() -> { Thread.sleep(100); return "fast-server"; });

        scope.join();
        String winner = scope.result(); // returns "fast-server"
        System.out.println("Winner: " + winner);
    }
}
```

---

## 9. Exercises

### Exercise 1 – Race Condition Fix
The following class has a race condition. Fix it using `synchronized` and then with `AtomicInteger`:
```java
class Counter {
    private int value = 0;
    public void increment() { value++; }
    public int get() { return value; }
}
```
Run 10 threads each incrementing 1000 times and verify the final count is always 10,000.

### Exercise 2 – Thread Pool HTTP Simulator
Simulate fetching 100 URLs using:
1. `newFixedThreadPool(10)` with platform threads
2. `newVirtualThreadPerTaskExecutor()` with virtual threads

Each "fetch" sleeps 50ms. Compare total execution time.

### Exercise 3 – Producer-Consumer
Implement a producer-consumer with:
- A `BlockingQueue<Integer>` of capacity 20
- 3 producers, each producing 100 items (random integers, 10ms delay each)
- 5 consumers, each consuming until they receive a `-1` sentinel value
- Ensure all items are processed and print the total sum

### Exercise 4 – CompletableFuture Pipeline
Build an async pipeline that:
1. Fetches a user by ID (simulate with `Thread.sleep(200)`)
2. Fetches the user's orders in parallel with fetching their address
3. Combines results into a `UserProfile` record
4. Handles failures gracefully

### Exercise 5 – Deadlock Detection
Create a deliberate deadlock situation with two threads and two locks. Then fix it by enforcing a consistent lock-acquisition order.

---

> ⬅️ Previous: [Module 11 – Java I/O & NIO](11-io-and-nio.md) | ➡️ Next: [Module 13 – Java 23 New Features](13-java23-new-features.md)
