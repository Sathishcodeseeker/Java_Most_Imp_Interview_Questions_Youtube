Below is a **carefully curated set of GENERAL Java interview questions** that are **not tied to a single feature (like GC or Collections)** but are **routinely asked across experience levels**.
Each question includes **what the interviewer is testing** and a **deep explanation**, aligned with **Java 8–17 and real-world systems**.

This is the **“everything else” bucket** that decides interviews.

---

# GENERAL JAVA INTERVIEW QUESTIONS (WITH DEEP EXPLANATION)

---

## PART 1 — Java Language Fundamentals (Non-Negotiable)

---

## 1. Is Java pass-by-value or pass-by-reference?

### Correct answer (very important)

> **Java is always pass-by-value.**

### Why people get confused

* The *value* passed for objects is a **reference value**

```java
void change(Person p) {
    p = new Person(); // reassigns local copy
}

Person p1 = new Person();
change(p1);
// p1 unchanged
```

### Key explanation

* Reference itself is copied
* Caller reference is not modified

**What interviewer tests**

* Memory model clarity
* Reference vs object distinction

---

## 2. Difference between `==` and `equals()`

### `==`

* Compares **references** (memory identity)

### `equals()`

* Compares **logical equality** (can be overridden)

```java
String a = new String("x");
String b = new String("x");

a == b       // false
a.equals(b)  // true
```

**Interview trap**

> Overriding `equals()` without `hashCode()` breaks collections.

---

## 3. Why is `String` immutable?

### Reasons (all expected)

1. Thread safety
2. HashMap key safety
3. Security (class loaders, paths)
4. String pool reuse

```java
String s = "abc";
s.concat("d"); // creates new object
```

---

## 4. Difference between `String`, `StringBuilder`, `StringBuffer`

| Type          | Mutable | Thread-Safe | Use case    |
| ------------- | ------- | ----------- | ----------- |
| String        | ❌       | ✅           | Constants   |
| StringBuilder | ✅       | ❌           | Performance |
| StringBuffer  | ✅       | ✅           | Legacy      |

**Interview insight**

> StringBuffer is almost never used in modern code.

---

## PART 2 — OOP & Design (Very High Signal)

---

## 5. Difference between abstraction and encapsulation

### Abstraction

* Hiding **what** is done
* Interfaces / abstract classes

### Encapsulation

* Hiding **how** it’s done
* Private fields + controlled access

Interviewers want **clear distinction**, not buzzwords.

---

## 6. Can we override a static method?

❌ No.

Static methods are:

* Compile-time bound
* Class-level

This is **method hiding**, not overriding.

---

## 7. Why composition is preferred over inheritance?

### Inheritance problems

* Tight coupling
* Fragile base class
* Hard to evolve

### Composition advantages

* Flexible
* Replaceable behavior
* Better testability

**Interview phrase**

> “Inheritance models *is-a*, composition models *has-a*.”

---

## 8. What is the diamond problem and how Java avoids it?

Java avoids it by:

* Allowing **multiple inheritance only for interfaces**
* Requiring **explicit override** when conflict exists

```java
interface A { default void f() {} }
interface B { default void f() {} }

class C implements A, B {
    public void f() {
        A.super.f();
    }
}
```

---

## PART 3 — Exceptions (Very Common)

---

## 9. Checked vs unchecked exceptions (modern view)

### Checked

* Compile-time enforced
* Often avoided in APIs

### Unchecked

* Runtime exceptions
* Preferred for business logic

**Senior insight**

> Overusing checked exceptions leads to brittle APIs.

---

## 10. Why `finally` can be dangerous

```java
try {
    return 1;
} finally {
    return 2;
}
```

Finally **overrides return** → bug.

Also:

* Can suppress original exception

---

## PART 4 — Concurrency (Beyond Basics)

---

## 11. Difference between process and thread

| Process         | Thread               |
| --------------- | -------------------- |
| Heavy           | Lightweight          |
| Separate memory | Shared memory        |
| IPC costly      | Faster communication |

---

## 12. What is deadlock?

Four conditions (must know):

1. Mutual exclusion
2. Hold and wait
3. No preemption
4. Circular wait

Removing **any one** prevents deadlock.

---

## 13. What is ThreadLocal and why is it dangerous?

### What it does

* Stores data **per thread**

### Danger

* Memory leaks in thread pools
* Value lives longer than request

Common production issue.

---

## PART 5 — JVM & Runtime (General)

---

## 14. Difference between JDK, JRE, JVM

| Component | Purpose           |
| --------- | ----------------- |
| JVM       | Executes bytecode |
| JRE       | JVM + libraries   |
| JDK       | JRE + dev tools   |

---

## 15. What is class loading?

Phases:

1. Loading
2. Linking
3. Initialization

Custom classloaders can cause:

* Metaspace leaks
* Class version conflicts

---

## 16. What is JIT compiler?

* Converts bytecode → native code
* Optimizes hot paths
* Improves runtime performance

Explains:

* JVM warm-up
* Why first requests are slower

---

## PART 6 — Java 8+ Functional Concepts

---

## 17. Difference between `map()` and `flatMap()`

```java
Stream<List<String>> → map
Stream<String> → flatMap
```

FlatMap **flattens nested structures**.

---

## 18. Why Optional should not be used as a field

Optional is:

* Return-type tool
* Not a data container

Using it as a field:

* Breaks serialization
* Adds overhead

---

## PART 7 — Immutability & Safety

---

## 19. Difference between final, finally, finalize

| Keyword  | Meaning              |
| -------- | -------------------- |
| final    | Prevent change       |
| finally  | Always executes      |
| finalize | GC hook (deprecated) |

---

## 20. Why immutability improves concurrency

* No synchronization
* No race conditions
* Safe sharing across threads

This connects to:

* Records
* Functional style
* Microservices safety

---

## PART 8 — Practical / Judgment Questions (Senior Level)

---

## 21. When would you NOT use Java Streams?

* Heavy loops
* Performance-critical paths
* Complex debugging needs

---

## 22. What Java feature do you avoid in production?

Good answers:

* `finalize()`
* Deep inheritance
* Over-engineering with streams

They test **judgment**, not correctness.

---

## 23. What happens if an exception is thrown in a thread?

* Thread dies
* JVM continues
* Unless main thread → JVM exits

---

## FINAL INTERVIEW-READY SUMMARY

> “Java is a strongly typed, garbage-collected, object-oriented language with a clear runtime model: threads execute with private stacks over a shared heap, objects are reference-based, immutability improves safety, and modern Java emphasizes clarity, correctness, and concurrency over low-level memory control.”

---


