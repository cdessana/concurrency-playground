# Concurrency Playground: Grounded in Java Concurrency in Practice (JCIP)

Welcome to the **Concurrency Playground**, an interactive, executable codebase designed to transform the complex theories of multithreading into tangible, reproducible code. 

This repository serves as a companion workbook mapping directly to the design patterns, safety principles, and classic pitfalls detailed in the industry-standard text, ***Java Concurrency in Practice* (JCIP)** by Brian Goetz, Tim Peierls, Joshua Bloch, Joseph Bowbeer, David Holmes, and Doug Lea.

Instead of just reading about race conditions, this project allows you to **execute them, trigger failures, and verify the thread-safe remedies** using modern Java and JUnit 5 stress-test harnesses.

---

## 🗺️ Curriculum & Repository Roadmap

The repository is structured as a multi-module Maven project. Each module contains a "bad" implementation (to demonstrate the hazard), a "remedy" implementation (showing the safe pattern), and JUnit tests that programmatically prove the differences.

### [Module 1: Thread Safety & Atomicity](./01-thread-safety)
* **Concepts:** Race conditions, read-modify-write compound actions, data races, and instruction interleaving.
* **Key Listings:**
  * `UnsafeCounter.java` (Inspired by *Listing 2.2 UnsafeCountingFactorizer*) — A raw `long` counter incremented with `++count`, which loses counts under multi-threaded load.
  * `AtomicCounter.java` (Inspired by *Listing 2.4 CountingFactorizer*) — Safe, lock-free delegation utilizing `AtomicLong`.
  * `SynchronizedCounter.java` (Inspired by *Listing 1.2 Sequence*) — Thread safety via intrinsic locking (`synchronized`).
* **The Proof:** `UnsafeCounterTest.java` launches parallel threads using an `ExecutorService` and `CountDownLatch` to show how counts are systematically dropped in the unsafe version, while the atomic and synchronized versions always reach 100% accuracy.

### [Module 2: Sharing Objects & Visibility](./02-sharing-objects)
* **Concepts:** Memory visibility, stale data, non-atomic 64-bit operations (long/double), thread confinement, and immutability.
* **Key Listings:**
  * `NoVisibilityDemo.java` (Inspired by *Listing 3.1 NoVisibility*) — Illustrates how one thread can loop forever because it cannot "see" a boolean flag update written by another thread due to compiler/CPU optimizations.
  * `ThreadConfinementDemo.java` (Chapter 3) — Shows how to isolate state safely using `ThreadLocal` and Stack Confinement, bypassing the need for synchronization entirely.
  * `ImmutableObjectDemo.java` (Chapter 3) — Demonstrates why immutable objects are inherently thread-safe and can be shared freely without locking.

### [Module 3: Application Building Blocks](./03-building-blocks)
* **Concepts:** State-dependent blocking, bounded queues, and synchronizers (`CountDownLatch`, `FutureTask`, `Semaphore`, `CyclicBarrier`).
* **Key Listings:**
  * `TestHarness.java` (Inspired by *Listing 5.11*) — A generic timing harness that utilizes latches to start and stop a given number of concurrent tasks simultaneously.

### [Module 4: The Ultimate Interview Capstone — Scalable Result Cache](./04-building-a-scalable-cache)
* **Concepts:** Thread-safe mapping, preventing duplicate computations, and leveraging `Future` as a promise.
* **The Progression (Chapter 5, Section 5.6):**
  * `Memorizer1` — Very slow. Wraps a standard `HashMap` with class-level `synchronized` locks, bottlenecking all threads.
  * `Memorizer2` — Uses `ConcurrentHashMap` to allow parallel reads, but suffers from duplicate computation bugs when multiple threads request the same uncomputed key.
  * `Memorizer3` — Switches to `ConcurrentHashMap<K, Future<V>>` to cache the *computation promise* rather than the result, significantly narrowing the race condition.
  * `Memorizer` — Synthesizes `ConcurrentMap.putIfAbsent` and `FutureTask` to build a flawless, fully-scalable result cache with zero duplicate computations.

### [Module 5: Liveness & Performance Hazards](./05-liveness-hazards)
* **Concepts:** Deadlocks, lock-ordering bugs, lock contention, and automated deadlock detection.
* **Key Listings:**
  * `DynamicDeadlockDemo.java` (Inspired by *Listing 10.2 DynamicLockOrderDeadlock*) — A simulated banking system showing how transferring money between accounts can trigger a deadlock if locks are acquired in different orders depending on input parameters.
  * `DeadlockRemedy.java` — Fixing the deadlock using ordered resource keys (e.g., `System.identityHashCode`) or explicit `tryLock()` timeouts.

---

## 🛠️ Getting Started & Running the Tests

To compile the codebase and run the stress tests locally, ensure you have **Java 17 (or higher)** and **Maven 3.8+** installed.

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/concurrency-playground.git
cd concurrency-playground
```

### 2. Build the Project
Compile the modules and download dependencies:
```bash
mvn clean compile
```

### 3. Run the Concurrency Tests
Execute the entire test suite. Pay close attention to the terminal output: you will see JUnit reports detailing exactly how and why the thread-unsafe code blocks fail under parallel execution, while the thread-safe blocks pass!
```bash
mvn test
```

---

## 📚 Grounding & Design Philosophy

This repository is built around the fundamental axiom of Java concurrency:

> "Writing thread-safe code is, at its core, about managing access to **state**, and in particular to **shared, mutable state**." 
> — *Brian Goetz, Java Concurrency in Practice*

Each module's codebase is designed to be highly readable, with detailed JavaDoc comments citing relevant page numbers and listing figures from the book. By exploring these files, you will transition from a programmer who writes concurrent code that works "by accident" to a craftsman who writes concurrent applications that are **predictably correct, resilient, and highly performant**.
