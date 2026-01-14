Below are the **TOP 20 Spring Boot annotations** you **must know and actually use** in real projects and interviews.
This list is **production-focused**, not exhaustive.

---

## 1. Core Bootstrapping

### 1️⃣ `@SpringBootApplication`

* Entry point of Spring Boot
* Combines:

  * `@Configuration`
  * `@EnableAutoConfiguration`
  * `@ComponentScan`

```java
@SpringBootApplication
public class OrderApplication { }
```

---

## 2. Stereotype Annotations (Bean Creation)

### 2️⃣ `@Component`

* Generic Spring-managed bean

### 3️⃣ `@Service`

* Business logic layer
* Semantically marks intent (important for readability & AOP)

### 4️⃣ `@Repository`

* Data access layer
* Enables exception translation (`SQLException → DataAccessException`)

### 5️⃣ `@Controller`

* MVC controller (returns views)

### 6️⃣ `@RestController`

* `@Controller + @ResponseBody`
* Used for REST APIs

```java
@RestController
class OrderController { }
```

---

## 3. Dependency Injection

### 7️⃣ `@Autowired`

* Injects dependencies
* Prefer **constructor injection**

```java
public OrderService(Repo repo) { }
```

### 8️⃣ `@Qualifier`

* Resolve multiple bean ambiguity

```java
@Qualifier("upiProcessor")
```

### 9️⃣ `@Value`

* Inject values from config

```java
@Value("${timeout.ms}")
private int timeout;
```

---

## 4. Configuration & Properties

### 🔟 `@Configuration`

* Java-based config class

### 1️⃣1️⃣ `@Bean`

* Explicit bean definition

```java
@Bean
ObjectMapper mapper() { }
```

### 1️⃣2️⃣ `@ConfigurationProperties`

* Strongly-typed config binding (preferred over `@Value`)

```java
@ConfigurationProperties(prefix="app")
```

---

## 5. Web / REST Layer

### 1️⃣3️⃣ `@RequestMapping`

* Base mapping (class or method)

### 1️⃣4️⃣ `@GetMapping`

### 1️⃣5️⃣ `@PostMapping`

### 1️⃣6️⃣ `@PutMapping`

### 1️⃣7️⃣ `@DeleteMapping`

```java
@GetMapping("/orders/{id}")
```

### 1️⃣8️⃣ `@RequestBody`

* JSON → Java object

### 1️⃣9️⃣ `@PathVariable`

* URI variable binding

### 2️⃣0️⃣ `@RequestParam`

* Query parameters

---

## 6. Transactions & AOP (CRITICAL)

### ⭐ `@Transactional`

* Enables transaction boundary
* Implemented via **Spring Proxy (AOP)**

```java
@Transactional
public void placeOrder() { }
```

**Interview fact:**
Does NOT work on:

* `private` methods
* Self-invocation

---

## 7. Exception Handling (Very Important)

### ⭐ `@ExceptionHandler`

* Local exception handling

### ⭐ `@ControllerAdvice`

* Global exception handling

```java
@ControllerAdvice
class GlobalExceptionHandler { }
```

---

## 8. Validation

### ⭐ `@Valid`

* Triggers bean validation

### ⭐ `@NotNull`, `@NotBlank`, `@Size`

* Field-level validation

```java
@NotBlank
private String name;
```

---

## 9. Conditional & Profiles (Production Use)

### ⭐ `@Profile`

* Environment-based beans

```java
@Profile("prod")
```

### ⭐ `@ConditionalOnProperty`

* Enable bean based on config

---

## 10. Caching, Async, Scheduling (Common in Prod)

### ⭐ `@Cacheable`

### ⭐ `@CacheEvict`

### ⭐ `@Async`

* Runs method asynchronously

### ⭐ `@Scheduled`

* Cron jobs

---

## 🔴 If You Are Interviewing for Senior Roles

You **must be able to explain**:

* How `@Transactional` works internally (proxy)
* Difference between `@Component` vs `@Service`
* Why `@ConfigurationProperties` is better than `@Value`
* Why constructor injection is preferred
* Why annotations fail in self-invocation

---

## Minimal “Must-Master” Subset (If Time Is Limited)

1. `@SpringBootApplication`
2. `@RestController`
3. `@Service`
4. `@Repository`
5. `@Autowired`
6. `@Configuration`
7. `@Bean`
8. `@Transactional`
9. `@ControllerAdvice`
10. `@GetMapping`

---

If you want next:

* **Annotation → internal working (AOP, proxies)**
* **Tricky interview questions on annotations**
* **Common annotation misuse in production**
* **Spring Boot annotations vs Java EE annotations**

Say which one.
