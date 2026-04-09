# Module 07 – Collections Framework

## Table of Contents
1. [Overview](#1-overview)
2. [List](#2-list)
3. [Set](#3-set)
4. [Map](#4-map)
5. [Queue and Deque](#5-queue-and-deque)
6. [Utility Classes: Collections & Arrays](#6-utility-classes-collections--arrays)
7. [Choosing the Right Collection](#7-choosing-the-right-collection)
8. [Immutable Collections (Java 9+)](#8-immutable-collections-java-9)
9. [Sequenced Collections (Java 21+)](#9-sequenced-collections-java-21)
10. [Exercises](#10-exercises)

---

## 1. Overview

The **Java Collections Framework** provides a unified architecture for storing and manipulating groups of objects.

```
Collection (interface)
├── List       (ordered, allows duplicates)
│   ├── ArrayList
│   ├── LinkedList
│   └── Vector (legacy)
├── Set        (no duplicates)
│   ├── HashSet
│   ├── LinkedHashSet
│   └── TreeSet (sorted)
└── Queue      (FIFO)
    ├── LinkedList
    ├── PriorityQueue
    └── Deque (double-ended)
        └── ArrayDeque

Map (interface – not part of Collection hierarchy)
├── HashMap
├── LinkedHashMap
├── TreeMap (sorted)
└── Hashtable (legacy)
```

---

## 2. List

`List` maintains **insertion order** and allows **duplicate** elements.

### ArrayList – best for random access, poor for insertions/deletions in the middle
```java
import java.util.ArrayList;
import java.util.List;

List<String> fruits = new ArrayList<>();
fruits.add("Apple");
fruits.add("Banana");
fruits.add("Cherry");
fruits.add("Banana");     // duplicates allowed

System.out.println(fruits);          // [Apple, Banana, Cherry, Banana]
System.out.println(fruits.get(1));   // Banana
System.out.println(fruits.size());   // 4
System.out.println(fruits.contains("Cherry")); // true

fruits.set(1, "Blueberry");          // replace at index
fruits.remove("Banana");             // removes first occurrence
fruits.remove(0);                    // removes by index

// Iterating
for (String fruit : fruits) {
    System.out.print(fruit + " ");
}
// Using iterator
var it = fruits.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

// Sort
java.util.Collections.sort(fruits);
fruits.sort(java.util.Comparator.naturalOrder());

// Sublist
List<String> sub = fruits.subList(0, 2); // view, not copy
```

### LinkedList – best for frequent insertions/deletions
```java
import java.util.LinkedList;

LinkedList<Integer> nums = new LinkedList<>(List.of(1, 2, 3, 4, 5));
nums.addFirst(0);     // add at beginning
nums.addLast(6);      // add at end
nums.removeFirst();   // remove from beginning
nums.removeLast();    // remove from end

System.out.println(nums); // [1, 2, 3, 4, 5]
```

---

## 3. Set

`Set` does **not** allow duplicate elements.

### HashSet – fastest, no guaranteed order
```java
import java.util.HashSet;
import java.util.Set;

Set<String> colors = new HashSet<>();
colors.add("red");
colors.add("green");
colors.add("blue");
colors.add("red");       // duplicate – ignored

System.out.println(colors.size());      // 3
System.out.println(colors.contains("green")); // true

// Set operations
Set<Integer> a = new HashSet<>(Set.of(1, 2, 3, 4));
Set<Integer> b = new HashSet<>(Set.of(3, 4, 5, 6));

// Union
Set<Integer> union = new HashSet<>(a);
union.addAll(b);       // {1, 2, 3, 4, 5, 6}

// Intersection
Set<Integer> intersection = new HashSet<>(a);
intersection.retainAll(b); // {3, 4}

// Difference
Set<Integer> diff = new HashSet<>(a);
diff.removeAll(b);     // {1, 2}
```

### LinkedHashSet – insertion-order preserved
```java
import java.util.LinkedHashSet;
Set<String> ordered = new LinkedHashSet<>();
ordered.add("banana");
ordered.add("apple");
ordered.add("cherry");
System.out.println(ordered); // [banana, apple, cherry]
```

### TreeSet – sorted order
```java
import java.util.TreeSet;
TreeSet<Integer> sorted = new TreeSet<>(Set.of(5, 2, 8, 1, 9));
System.out.println(sorted);          // [1, 2, 5, 8, 9]
System.out.println(sorted.first());  // 1
System.out.println(sorted.last());   // 9
System.out.println(sorted.headSet(5)); // [1, 2]
System.out.println(sorted.tailSet(5)); // [5, 8, 9]
```

---

## 4. Map

`Map` stores **key-value pairs**. Keys must be unique; values may duplicate.

### HashMap – fastest, unordered
```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob",   82);
scores.put("Carol", 90);
scores.put("Alice", 97);     // overwrites existing key

System.out.println(scores.get("Alice"));       // 97
System.out.println(scores.getOrDefault("Dave", 0)); // 0
System.out.println(scores.containsKey("Bob")); // true
System.out.println(scores.size());             // 3

// Iterating
for (Map.Entry<String, Integer> entry : scores.entrySet()) {
    System.out.println(entry.getKey() + " → " + entry.getValue());
}

// Java 8+ methods
scores.forEach((name, score) -> System.out.println(name + ": " + score));

scores.putIfAbsent("Eve", 88);    // add only if key absent
scores.computeIfAbsent("Frank", k -> k.length() * 10); // compute value

// Merge
scores.merge("Alice", 3, Integer::sum); // Alice: 97 + 3 = 100
```

### LinkedHashMap – insertion-order preserved
```java
import java.util.LinkedHashMap;
Map<String, String> capitals = new LinkedHashMap<>();
capitals.put("Brazil",  "Brasília");
capitals.put("France",  "Paris");
capitals.put("Japan",   "Tokyo");
// Iteration preserves insertion order
```

### TreeMap – sorted by key
```java
import java.util.TreeMap;
TreeMap<String, Integer> tree = new TreeMap<>(scores);
System.out.println(tree.firstKey()); // alphabetically first key
System.out.println(tree.lastKey());  // alphabetically last key
System.out.println(tree.headMap("Carol")); // entries before "Carol"
```

---

## 5. Queue and Deque

### Queue – FIFO (First In, First Out)
```java
import java.util.Queue;
import java.util.LinkedList;

Queue<String> queue = new LinkedList<>();
queue.offer("first");
queue.offer("second");
queue.offer("third");

System.out.println(queue.peek());    // "first" (doesn't remove)
System.out.println(queue.poll());    // "first" (removes)
System.out.println(queue.size());    // 2
```

### PriorityQueue – elements ordered by priority
```java
import java.util.PriorityQueue;

PriorityQueue<Integer> pq = new PriorityQueue<>(); // min-heap by default
pq.offer(5);
pq.offer(1);
pq.offer(3);

while (!pq.isEmpty()) {
    System.out.print(pq.poll() + " "); // 1 3 5
}

// Max-heap with comparator
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(java.util.Comparator.reverseOrder());
```

### ArrayDeque – double-ended queue (also best Stack replacement)
```java
import java.util.ArrayDeque;
import java.util.Deque;

Deque<String> deque = new ArrayDeque<>();
deque.addFirst("middle");
deque.addFirst("first");
deque.addLast("last");

System.out.println(deque.peekFirst()); // first
System.out.println(deque.peekLast());  // last

// Use as a Stack
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
System.out.println(stack.pop()); // 3 (LIFO)
```

---

## 6. Utility Classes: Collections & Arrays

```java
import java.util.Collections;
import java.util.Arrays;

List<Integer> list = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2, 6));

Collections.sort(list);                          // [1, 1, 2, 3, 4, 5, 6, 9]
Collections.reverse(list);                       // [9, 6, 5, 4, 3, 2, 1, 1]
Collections.shuffle(list);                       // random order
Collections.swap(list, 0, 1);                   // swap elements at index 0 and 1
System.out.println(Collections.min(list));       // minimum
System.out.println(Collections.max(list));       // maximum
System.out.println(Collections.frequency(list, 1)); // count occurrences of 1

// Binary search (requires sorted list)
Collections.sort(list);
int idx = Collections.binarySearch(list, 5);    // index of 5

// Arrays utilities
int[] arr = {5, 3, 1, 4, 2};
Arrays.sort(arr);                                // [1, 2, 3, 4, 5]
System.out.println(Arrays.toString(arr));        // [1, 2, 3, 4, 5]
int[][] matrix = {{1, 2}, {3, 4}};
System.out.println(Arrays.deepToString(matrix)); // [[1, 2], [3, 4]]

// Convert array ↔ list
String[] strArr = {"a", "b", "c"};
List<String> fromArray = Arrays.asList(strArr);  // fixed-size list
List<String> mutable = new ArrayList<>(Arrays.asList(strArr));
String[] backToArray = mutable.toArray(new String[0]);
```

---

## 7. Choosing the Right Collection

| Need | Use |
|------|-----|
| Ordered list, random access | `ArrayList` |
| Ordered list, many inserts/deletes at ends | `LinkedList` |
| No duplicates, fast lookup | `HashSet` |
| No duplicates, sorted | `TreeSet` |
| No duplicates, insertion order | `LinkedHashSet` |
| Key-value pairs, fast lookup | `HashMap` |
| Key-value pairs, sorted by key | `TreeMap` |
| Key-value pairs, insertion order | `LinkedHashMap` |
| FIFO queue | `ArrayDeque` |
| Priority-based queue | `PriorityQueue` |
| Stack (LIFO) | `ArrayDeque` (prefer over `Stack`) |

---

## 8. Immutable Collections (Java 9+)

```java
// List.of, Set.of, Map.of – immutable, no null allowed
List<String> immutableList = List.of("a", "b", "c");
Set<Integer> immutableSet  = Set.of(1, 2, 3);
Map<String, Integer> immutableMap = Map.of("one", 1, "two", 2);

// Map.entry for more than 10 entries
Map<String, Integer> bigMap = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2),
    Map.entry("c", 3)
);

// immutableList.add("d"); // UnsupportedOperationException!

// copyOf – immutable copy of a mutable collection
List<String> mutable = new ArrayList<>(List.of("x", "y"));
List<String> copy = List.copyOf(mutable); // snapshot
```

---

## 9. Sequenced Collections (Java 21+)

Java 21 introduced `SequencedCollection`, `SequencedSet`, and `SequencedMap` interfaces providing uniform first/last element access:

```java
import java.util.SequencedCollection;

List<String> list = new ArrayList<>(List.of("first", "middle", "last"));
SequencedCollection<String> sc = list;

System.out.println(sc.getFirst()); // first
System.out.println(sc.getLast());  // last

sc.addFirst("new-first");
sc.addLast("new-last");
sc.removeFirst();
sc.removeLast();

// Reversed view
for (String s : sc.reversed()) {
    System.out.print(s + " ");
}

// SequencedMap
import java.util.LinkedHashMap;
import java.util.SequencedMap;
SequencedMap<String, Integer> smap = new LinkedHashMap<>();
smap.put("a", 1);
smap.put("b", 2);
smap.put("c", 3);
System.out.println(smap.firstEntry()); // a=1
System.out.println(smap.lastEntry());  // c=3
```

---

## 10. Exercises

### Exercise 1 – Word Frequency Counter
Read a sentence from the user and count how many times each word appears. Print the words sorted alphabetically with their counts.

### Exercise 2 – Unique Characters
Write a method that takes a `String` and returns a `Set<Character>` of its unique characters, preserving insertion order.

### Exercise 3 – Phone Book
Implement a simple phone book using `TreeMap<String, String>` (name → phone number). Support:
- Add contact
- Remove contact
- Look up by name
- Print all contacts sorted by name

### Exercise 4 – Task Queue
Simulate a task queue with `ArrayDeque<String>`:
- `addTask(String task)` – add to the back
- `processNext()` – remove and print from the front
- `preempt(String urgentTask)` – add to the front
- Process all tasks

### Exercise 5 – Student Grades
Given a list of students (name, grade), use streams or collection methods to:
1. Find the student with the highest grade
2. Calculate the average grade
3. Group students by letter grade (A, B, C, D, F)
4. List all students sorted by grade descending

### Exercise 6 – Cache with LRU Eviction
Use `LinkedHashMap` to implement a simple LRU (Least Recently Used) cache with a fixed capacity. Override `removeEldestEntry` to evict the oldest entry when capacity is exceeded.

---

> ⬅️ Previous: [Module 06 – Interfaces & Abstract Classes](06-interfaces-and-abstract-classes.md) | ➡️ Next: [Module 08 – Generics](08-generics.md)
