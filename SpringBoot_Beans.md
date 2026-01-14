Below is a **0 → 10 incremental explanation of Spring Beans**, written for **someone starting from zero**, with **code at every layer**.
Read it **top to bottom** — each layer builds on the previous one.

---

## Layer 0️⃣ — What is a Spring Bean? (Absolute zero)

**Plain meaning**

> A **Spring Bean** is just a **normal Java object** that is **created and managed by Spring**, not by you.

Without Spring:

```java
UserService service = new UserService();
```

With Spring:

```java
UserService service = springGivesYouThis();
```

Spring handles:

* creation
* lifetime
* sharing
* destruction

This is done by the **Spring Framework**.

---

## Layer 1️⃣ — Why do we need Spring Beans?

### Problem without Spring

```java
public class OrderController {
    private OrderService service = new OrderService();
}
```

Problems:

* Tight coupling
* Hard to test
* Hard to change implementation
* Manual object creation everywhere

### Solution with Spring

Spring creates objects **once**, **reuses**, and **injects** them.

---

## Layer 2️⃣ — The Container (very important concept)

Spring has a **container** (think of it as a factory).

Container responsibilities:

* create beans
* store beans
* give beans when needed

You never call `new` for beans.

```text
Spring Container
 ├── UserService
 ├── OrderService
 └── PaymentService
```

---

## Layer 3️⃣ — Creating your first Bean (`@Component`)

```java
@Component
public class UserService {
    public String getUser() {
        return "Sathish";
    }
}
```

What happens:

* Spring scans classpath
* Finds `@Component`
* Creates object
* Stores it in container

You did **NOT** write `new UserService()`.

---

## Layer 4️⃣ — Using a Bean (`@Autowired`)

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/user")
    public String user() {
        return userService.getUser();
    }
}
```

What Spring does:

1. Finds `UserService` bean
2. Injects it into `UserController`
3. No manual wiring

This is called **Dependency Injection (DI)**.

---

## Layer 5️⃣ — Bean stereotypes (`@Component` family)

All these create beans:

```java
@Component   // generic
@Service     // business logic
@Repository  // database layer
@Controller  // MVC controller
```

Example:

```java
@Service
public class PaymentService { }
```

They behave the **same**, but improve readability and tooling.

---

## Layer 6️⃣ — Bean Scope (how long a bean lives)

Default scope is **singleton**.

```java
@Component
public class AppConfig { }
```

Meaning:

* ONE object for entire application
* Shared by all requests

Other scopes:

```java
@Scope("request")    // per HTTP request
@Scope("session")    // per user session
@Scope("prototype")  // new object every time
```

Example:

```java
@Component
@Scope("request")
public class RequestContext { }
```

---

## Layer 7️⃣ — Bean Lifecycle (birth → death)

Spring bean lifecycle steps:

1. Constructor
2. Dependency injection
3. Initialization
4. Ready to use
5. Destruction

```java
@Component
public class CacheService {

    @PostConstruct
    public void init() {
        System.out.println("Bean created");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Bean destroyed");
    }
}
```

Spring automatically calls these.

---

## Layer 8️⃣ — Configuration-based Beans (`@Bean`)

Used when:

* class is from third-party library
* you cannot add annotations

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

Now `ObjectMapper` is a Spring Bean.

---

## Layer 9️⃣ — How Spring finds Beans (Component Scanning)

```java
@SpringBootApplication
public class MyApp { }
```

This does:

```text
@ComponentScan
@EnableAutoConfiguration
@Configuration
```

Spring scans:

* current package
* sub-packages

Anything with `@Component` is registered automatically.

---

## Layer 🔟 — How Beans relate to memory & real systems

### Good Bean (stateless)

```java
@Service
public class PriceService {
    public int calculate(int price) {
        return price * 2;
    }
}
```

### Dangerous Bean (stateful singleton)

```java
@Service
public class BadService {
    private List<String> data = new ArrayList<>();
}
```

Why dangerous:

* singleton lives forever
* list grows
* memory leak

**Golden rule**

> Singleton beans must be stateless.

---

## One-page mental model (remember this)

```
Spring Bean =
  Java Object
  + created by Spring
  + managed by Spring
  + injected by Spring
  + lifecycle controlled by Spring
```

---

## Interview-ready 1-liner

> A Spring Bean is a Java object whose creation, lifecycle, scope, and dependencies are managed by the Spring container.

---

If you want next (recommended order):
1️⃣ Spring Bean vs Java Object
2️⃣ Bean vs Component vs Service
3️⃣ Bean scopes with memory-leak examples
4️⃣ How Spring internally creates beans
5️⃣ Bean proxying (AOP, transactions)

Tell me which one.
