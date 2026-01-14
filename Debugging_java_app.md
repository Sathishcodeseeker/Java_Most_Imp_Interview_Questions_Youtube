Below is a **practical, real-time debugging guide for Java production systems**, structured the way senior engineers actually use it. This covers **threads, dumps, profiling, and observability tools**, with *when*, *why*, and *how*—not just definitions.

---

## 1. Mental Model: What “Real-Time Debugging” Means in Java

In **production**, you usually **cannot attach a debugger**.
So you rely on **runtime signals**:

* Threads → *What is running / blocked / stuck*
* Dumps → *Snapshot of JVM internals*
* Profilers → *Where time & memory are actually going*
* Telemetry → *How requests flow across services*

Think of it as **observing a living system**, not stepping through code.

---

## 2. Threads: The First Thing to Check in Any Issue

### What is a Thread in Java (Production View)

* Each request / task runs on a thread
* Problems usually appear as:

  * Threads **BLOCKED**
  * Threads **WAITING**
  * Threads **RUNNABLE but slow**

### Common Thread Problems

| Symptom                   | Likely Cause                   |
| ------------------------- | ------------------------------ |
| CPU 100%                  | Infinite loop / busy spin      |
| Requests hanging          | Deadlock / lock contention     |
| Random slowness           | Thread pool exhaustion         |
| Works locally, fails prod | Blocking I/O on shared threads |

---

## 3. Thread Dump (MOST IMPORTANT TOOL)

A **thread dump** is a **snapshot of all threads + what they’re doing**.

![Image](https://www.baeldung.com/wp-content/uploads/2020/03/JVisualVM.png)

![Image](https://javaconceptoftheday.com/wp-content/uploads/2016/06/WaitingVsBlocked.png)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/22-2.png)

### How to Take a Thread Dump

```bash
jstack <PID>
```

or

```bash
kill -3 <PID>   # Linux (safe)
```

### What to Look For (Checklist)

1. **Thread State**

   * `RUNNABLE` → CPU usage
   * `BLOCKED` → Lock contention
   * `WAITING` / `TIMED_WAITING` → External dependency

2. **Repeated Stack Traces**

   * Same stack → Bottleneck
   * Same lock → Contention

3. **Deadlock Section**
   JVM explicitly tells you if deadlock exists.

### Real Production Pattern

> Take **3 thread dumps**, 10 seconds apart
> If same thread is stuck → **confirmed issue**

---

## 4. Heap Dump: Memory Problems (OOM, Leaks)

A **heap dump** captures **all objects in memory**.

![Image](https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/images/heapdump-classes-screen.png)

![Image](https://i.sstatic.net/nRrFi.png)

![Image](https://eclipse.dev/mat/images/mat_thumb.png)

### When to Take Heap Dump

* `OutOfMemoryError`
* Increasing memory with no GC relief
* Pods restarting due to memory

### How to Take

```bash
jmap -dump:format=b,file=heap.hprof <PID>
```

### What You Analyze

* Dominator tree
* Retained size
* Unexpected object growth (Maps, Lists, Caches)

### Tools

* Eclipse MAT
* VisualVM

---

## 5. Profiling: Where Time Is ACTUALLY Spent

Profiling answers:

> “Which method is slow and **why**?”

### Types of Profiling

| Type                 | Use           |
| -------------------- | ------------- |
| CPU profiling        | Slow requests |
| Memory profiling     | Leaks         |
| Allocation profiling | GC pressure   |

![Image](https://miro.medium.com/1%2A5c27xq5vCBvf7THW5hO7JQ.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2A5c27xq5vCBvf7THW5hO7JQ.png)

![Image](https://developer.android.com/static/studio/images/profiler-jk-allocations-visualization.png)

### Safe Production Profilers

* Java Flight Recorder (JFR)
* Async Profiler

---

## 6. Java Flight Recorder (JFR) – Production Gold Standard

JFR is:

* **Low overhead**
* Built into JVM
* Safe for prod

```bash
jcmd <PID> JFR.start
jcmd <PID> JFR.stop filename=recording.jfr
```

### What You Get

* CPU hotspots
* Lock contention
* GC pauses
* Thread scheduling

---

## 7. Garbage Collection (GC) Debugging

### Symptoms

| Symptom      | Meaning             |
| ------------ | ------------------- |
| Long pauses  | Stop-the-world      |
| Frequent GC  | Allocation pressure |
| High old gen | Memory leak         |

### Enable GC Logs

```bash
-Xlog:gc*
```

![Image](https://cdn.logviewplus.com/resources/19/assets/images/layout/video_gc.png)

![Image](https://i.sstatic.net/3IgvP.png)

---

## 8. Observability (Modern Mandatory Layer)

Thread dumps show **inside one JVM**
Observability shows **across services**

### Core Pillars

| Pillar  | Tool               |
| ------- | ------------------ |
| Logs    | ELK, Azure Monitor |
| Metrics | Prometheus         |
| Traces  | OpenTelemetry      |

### Traces Answer

* Which service is slow
* Which method inside service
* Which DB / external call

---

## 9. Real-World Debugging Playbook

### Scenario: Production Slowness

1. Check **metrics** (CPU, memory, latency)
2. Take **thread dumps**
3. Look for **BLOCKED / RUNNABLE loops**
4. Capture **JFR**
5. Correlate with **traces**
6. Fix code / config

---

## 10. Tool Summary (Must-Know)

| Category      | Tools               |
| ------------- | ------------------- |
| Thread dumps  | jstack, kill -3     |
| Heap dumps    | jmap                |
| Profiling     | JFR, Async Profiler |
| GC analysis   | GC logs             |
| Visual tools  | VisualVM, MAT       |
| Observability | OpenTelemetry       |

---

## 11. Key Senior-Level Insight

* **Thread dump** → “What is stuck?”
* **Profiler** → “Why is it slow?”
* **Heap dump** → “Why memory grows?”
* **Traces** → “Where is time lost across services?”

Below are **real Spring Boot memory-leak scenarios** mapped to the **top 5 causes**, with **production-style code**, **why it leaks**, and **how to fix it**.
All examples are **patterns seen in real services**, not toy cases.

---

## 1️⃣ Static collections holding request/business objects

### ❌ Leaky Spring Boot code

```java
@Component
public class UserCache {

    private static final Map<String, User> CACHE = new HashMap<>();

    public void add(User user) {
        CACHE.put(user.getId(), user); // grows forever
    }
}
```

### ❓ Why this leaks

* `static` → GC root
* Every request adds data
* No eviction, no lifecycle

### 🔍 Heap dump signature

```
GC Root → Class → static UserCache.CACHE → HashMap → User
```

### ✅ Fix

Use a **bounded cache** (Caffeine recommended).

```java
@Component
public class UserCache {

    private final Cache<String, User> cache =
        Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .build();

    public void add(User user) {
        cache.put(user.getId(), user);
    }
}
```

---

## 2️⃣ ThreadLocal leak in web request handling (VERY common)

### ❌ Leaky code

```java
public class RequestContext {
    public static final ThreadLocal<UserContext> CTX = new ThreadLocal<>();
}
```

```java
@Component
public class AuthFilter extends OncePerRequestFilter {

    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws IOException, ServletException {
        RequestContext.CTX.set(loadUser(req));
        chain.doFilter(req, res); // ❌ never cleared
    }
}
```

### ❓ Why this leaks

* Tomcat uses **thread pools**
* Threads live forever
* `ThreadLocal` values stay attached

### 🔍 Heap dump signature

```
Thread → ThreadLocalMap → UserContext (retained)
```

### ✅ Fix (MANDATORY)

```java
try {
    RequestContext.CTX.set(loadUser(req));
    chain.doFilter(req, res);
} finally {
    RequestContext.CTX.remove();
}
```

---

## 3️⃣ Unbounded in-memory cache / Map inside a service

### ❌ Leaky code

```java
@Service
public class PriceService {

    private final Map<String, BigDecimal> priceCache = new ConcurrentHashMap<>();

    public BigDecimal getPrice(String productId) {
        return priceCache.computeIfAbsent(productId, this::loadPrice);
    }
}
```

### ❓ Why this leaks

* One entry per product forever
* No TTL / size limit
* Looks harmless, kills prod slowly

### 🔍 Heap dump

* `ConcurrentHashMap` dominates heap
* Retained size grows steadily

### ✅ Fix

```java
@Service
public class PriceService {

    private final Cache<String, BigDecimal> cache =
        Caffeine.newBuilder()
                .maximumSize(50_000)
                .expireAfterAccess(5, TimeUnit.MINUTES)
                .build();

    public BigDecimal getPrice(String productId) {
        return cache.get(productId, this::loadPrice);
    }
}
```

---

## 4️⃣ Event listeners / callbacks never deregistered

### ❌ Leaky code

```java
@Component
public class OrderListener {

    @EventListener
    public void onOrder(OrderCreatedEvent event) {
        // holds references indirectly
    }
}
```

Now combined with **manual registration**:

```java
@Component
public class Startup {

    public Startup(ApplicationEventMulticaster multicaster,
                   OrderListener listener) {
        multicaster.addApplicationListener(listener); // ❌ never removed
    }
}
```

### ❓ Why this leaks

* Event multicaster holds strong reference
* Listener holds service + domain objects
* After redeploys → leak explodes

### 🔍 Heap dump

```
ApplicationEventMulticaster → Listener → Service → Domain objects
```

### ✅ Fix

* Let Spring manage listeners automatically
* Or deregister explicitly

```java
@PreDestroy
public void cleanup() {
    multicaster.removeApplicationListener(listener);
}
```

---

## 5️⃣ JPA / Hibernate session misuse (classic enterprise leak)

### ❌ Leaky code

```java
@Service
public class ReportService {

    @PersistenceContext
    private EntityManager em;

    public List<Order> generateReport() {
        return em.createQuery("from Order", Order.class).getResultList();
        // ❌ entities stay managed
    }
}
```

### ❓ Why this leaks

* Persistence context keeps references
* Large result sets = retained entities
* Especially bad in batch jobs

### 🔍 Heap dump

* `org.hibernate.engine.spi.PersistenceContext`
* Thousands of entity instances retained

### ✅ Fix

```java
@Transactional(readOnly = true)
public List<Order> generateReport() {
    List<Order> orders =
        em.createQuery("from Order", Order.class).getResultList();
    em.clear(); // or use pagination
    return orders;
}
```

Or better:

```java
Page<Order> page = repository.findAll(PageRequest.of(0, 1000));
```

---

## 🔴 Senior-level reality check (important)

> **Spring Boot itself rarely leaks.**
> Leaks happen when **developer-managed references outlive request scope**.

### Most dangerous combo in production

```
ThreadPool + ThreadLocal + Cache + Static reference
```

---

## 🧠 Interview-ready summary

| Cause       | Spring Boot Area    | Fix              |
| ----------- | ------------------- | ---------------- |
| Static refs | Utility / singleton | Bounded cache    |
| ThreadLocal | Filters, Security   | remove()         |
| Cache       | Services            | TTL + max size   |
| Listeners   | Events              | Deregister       |
| JPA         | Batch / reports     | Clear / paginate |

---

Below is a **step-by-step, real heap-dump leak walkthrough for a Spring Boot application**, exactly how this is done in **production incidents**. No theory padding.

![Image](https://i.sstatic.net/UFdvA.png)

![Image](https://help.eclipse.org/latest/topic/org.eclipse.mat.ui.help/mimes/leak_report.png)

![Image](https://i.sstatic.net/pi6pg.png)

![Image](https://i0.wp.com/blog.heaphero.io/wp-content/uploads/2025/05/Memory-Leak-Due-To-uncleared-threadlocal.png?fit=1200%2C627\&ssl=1)

---

## 1️⃣ Symptom observed in production (starting point)

Typical alert pattern:

* Heap usage keeps increasing
* Full GC runs but **old gen does NOT drop**
* Pod / JVM restarts with `OutOfMemoryError`

At this point, **metrics alone are not enough** → heap dump required.

---

## 2️⃣ Take heap dump at the correct moment (critical)

### ✅ Correct time

* When heap is **80–95% full**
* Just before restart / OOM

### ✅ Command

```bash
jmap -dump:format=b,file=heap.hprof <PID>
```

or enable upfront:

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

❌ Heap dump at low memory = **wasted effort**

---

## 3️⃣ Open heap dump in Eclipse MAT (industry standard)

First actions:

1. Open `heap.hprof`
2. Run **Leak Suspects Report**
3. Open **Dominator Tree**

Ignore noise. Focus on **retained size**, not shallow size.

---

## 4️⃣ Real leak case #1 — ThreadLocal leak (very common in Spring Boot)

### 🔍 What you see in Dominator Tree

Top dominators:

```
java.lang.Thread
 └── java.lang.ThreadLocal$ThreadLocalMap
      └── com.myapp.security.UserContext  (200MB)
```

### 🔎 Path to GC Root

```
Thread → ThreadLocalMap → UserContext → User → Permissions → Big collections
```

### ❌ Root cause code

```java
public class RequestContext {
    public static final ThreadLocal<UserContext> CTX = new ThreadLocal<>();
}
```

```java
CTX.set(userContext);
// ❌ remove() missing
```

### ✅ Fix

```java
try {
    CTX.set(userContext);
    chain.doFilter(req, res);
} finally {
    CTX.remove();
}
```

### 🧠 Why GC cannot collect

* Thread pool threads are **GC roots**
* ThreadLocal values stay attached forever

---

## 5️⃣ Real leak case #2 — Static cache in Spring singleton

### 🔍 Dominator Tree

```
java.util.concurrent.ConcurrentHashMap (1.5 GB retained)
 └── Order
 └── Customer
```

### 🔎 GC Root

```
Class → static field → Map
```

### ❌ Leaky code

```java
@Component
public class OrderCache {
    private static final Map<String, Order> CACHE = new ConcurrentHashMap<>();
}
```

### ✅ Fix

```java
Cache<String, Order> cache =
    Caffeine.newBuilder()
            .maximumSize(50_000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
```

---

## 6️⃣ Real leak case #3 — JPA Persistence Context explosion

### 🔍 Heap dump shows

```
org.hibernate.engine.internal.StatefulPersistenceContext
 └── Managed entities (hundreds of thousands)
```

### ❌ Leaky code

```java
List<Order> orders = em.createQuery("from Order").getResultList();
```

### Why it leaks

* All entities stay **managed**
* Persistence context retains references

### ✅ Fix

```java
List<Order> orders = em.createQuery("from Order").getResultList();
em.clear(); // or pagination
```

---

## 7️⃣ Real leak case #4 — Event listener retained after redeploy

### 🔍 Heap path

```
ApplicationEventMulticaster
 └── Listener
      └── Service
           └── Large domain objects
```

### Cause

* Manual listener registration
* No deregistration on shutdown

### Fix

* Let Spring manage lifecycle
* Or deregister in `@PreDestroy`

---

## 8️⃣ How to prove the fix (most teams skip this ❌)

### After fix deployment

1. Run same load
2. Take **another heap dump**
3. Compare with previous dump

### ✅ Healthy behavior

* Old gen stabilizes
* Dominator tree no longer shows growing objects
* GC frees memory after load drops

---

## 9️⃣ Production decision tree (use this mentally)

```
Old gen grows after GC?
 ├─ Yes → Heap dump
 │    ├─ Static root → Static leak
 │    ├─ Thread root → ThreadLocal leak
 │    ├─ PersistenceContext → JPA misuse
 │    └─ Cache → Eviction issue
 └─ No → Not a leak (capacity or traffic issue)
```

---

## 🔴 Senior-level truth (important)

> **99% of Spring Boot leaks are visible within 5 minutes of opening the Dominator Tree.**

Understood. You are **not asking about cache-miss logic**, but:

> **As a Spring Boot programmer, how do I avoid memory leaks when I do NOT use a cache at all?**

This is a **very good question** and shows the right mindset.

---

## Core truth (important)

> **Not having a cache does NOT automatically make your app safe from memory leaks.**
> Memory leaks happen when **objects live longer than the request/operation that created them** — cache or no cache.

So when **cache is absent**, you must be careful about **what replaces it implicitly**.

---

## The real risk when “cache is not present”

When you don’t use an explicit cache, leaks usually come from **implicit retention**, such as:

1. Static references
2. ThreadLocal misuse
3. JPA persistence context
4. Long-lived collections
5. Async / background tasks

Let’s go through **what to do instead**, concretely.

---

## 1️⃣ Do NOT replace cache with static collections (most common mistake)

### ❌ Dangerous “no-cache” replacement

```java
public class UserHolder {
    public static final List<User> users = new ArrayList<>();
}
```

Developers often do this thinking:

> “We don’t use cache, this is just temporary storage”

This **is a cache**, but **worse**:

* No eviction
* GC root
* Guaranteed memory leak

### ✅ Correct approach

If data is request-scoped:

```java
public List<User> handleRequest() {
    List<User> users = repository.findUsers();
    return users; // dies after request
}
```

**Rule**

> If cache is absent, data must stay **local to method or request scope**.

---

## 2️⃣ Be careful with JPA when no cache is used

Even without cache, **JPA itself retains objects**.

### ❌ Leak without cache

```java
@Transactional
public void processAll() {
    List<Order> orders = repository.findAll(); // huge
    orders.forEach(this::process);
}
```

Why this leaks:

* All entities are managed
* Persistence context grows
* Memory grows until transaction ends

### ✅ Leak-safe version

```java
@Transactional
public void processInBatches() {
    int page = 0;
    Page<Order> batch;

    do {
        batch = repository.findAll(PageRequest.of(page++, 500));
        batch.forEach(this::process);
        entityManager.clear(); // CRITICAL
    } while (!batch.isEmpty());
}
```

**Rule**

> No cache ≠ no retention. JPA *is* retention.

---

## 3️⃣ Avoid accidental “collection-as-cache” patterns

### ❌ Common accidental leak

```java
@Service
public class ReportService {

    private final List<Report> reports = new ArrayList<>();

    public void generate() {
        reports.add(createReport());
    }
}
```

This is an **implicit cache**.

### ✅ Correct

```java
public Report generate() {
    return createReport();
}
```

Or stream results directly:

```java
try (Stream<Order> orders = repository.streamAll()) {
    orders.forEach(this::process);
}
```

**Rule**

> If a collection lives beyond a method → treat it as a cache and design it.

---

## 4️⃣ ThreadLocal: safe if you didn’t create it

You said:

> “I never used ThreadLocal”

That’s good.

### What you should do

* **Don’t create custom ThreadLocal**
* **Don’t spawn raw threads**
* **Don’t assume ThreadLocal data exists in async code**

Spring cleans its own ThreadLocals safely.

### ❌ Leak happens only if you add this

```java
static ThreadLocal<Data> TL = new ThreadLocal<>();
```

**Rule**

> No cache + no ThreadLocal = much safer baseline.

---

## 5️⃣ Async / background tasks without cache

Even without cache, async code can leak memory.

### ❌ Leak-prone

```java
@Async
public void process() {
    List<Data> data = loadHugeData();
    // stored in field / static
}
```

### ✅ Leak-safe

```java
@Async
public void process() {
    loadHugeData().forEach(this::processOne);
}
```

**Rule**

> Async tasks must stream, not accumulate.

---

## 6️⃣ Logging & MDC cleanup (hidden leak)

Even without cache, MDC can retain data.

### ❌ Risky

```java
MDC.put("requestId", id);
// no cleanup
```

### ✅ Correct

```java
try {
    MDC.put("requestId", id);
    process();
} finally {
    MDC.clear();
}
```

---

## 7️⃣ Final mental model (this is the key)

When **cache is absent**, ask this question:

> “Where does this object live, and who releases it?”

If the answer is unclear → **potential leak**.

---

## Leak-safe rules when cache is NOT used

✔ Keep data method-scoped
✔ Never use static collections
✔ Stream large data, don’t collect
✔ Paginate JPA results
✔ Clear persistence context in loops
✔ Avoid custom ThreadLocal
✔ Don’t accumulate in async tasks

---

## One-line senior answer (interview-ready)

> *Even without using a cache, memory leaks can occur if objects escape request scope through static references, persistence contexts, or long-lived collections. The safest approach is to keep data method-scoped, stream large datasets, and explicitly manage lifecycles.*

---





