Below is a **production-oriented list of the MOST COMMON Spring Boot exceptions**, grouped by **where they occur**, **why they occur**, and **how to handle them**.
This is exactly what shows up in **real systems and interviews**.

---

## 1. Startup & Bean Creation Exceptions (VERY COMMON)

### 1️⃣ `BeanCreationException`

**Meaning:** Spring failed to create a bean
**Typical causes:**

* Missing dependency
* Exception in constructor
* Invalid config

**How to debug**

* Look at the **root cause** at the bottom of stack trace
* Usually wraps another exception

---

### 2️⃣ `NoSuchBeanDefinitionException`

**Meaning:** Spring couldn’t find a bean to inject

**Causes**

* Missing `@Component/@Service`
* Wrong `@ComponentScan`
* Wrong `@Profile`

```text
No qualifying bean of type X available
```

---

### 3️⃣ `UnsatisfiedDependencyException`

**Meaning:** Dependency injection failed

**Causes**

* Circular dependency
* Missing bean
* Multiple beans without `@Qualifier`

---

### 4️⃣ `BeanCurrentlyInCreationException`

**Meaning:** Circular dependency

```java
A → B → A
```

**Fix**

* Constructor injection + refactor
* Avoid circular design

---

## 2. Web / REST Layer Exceptions (Daily Use)

### 5️⃣ `HttpRequestMethodNotSupportedException`

**Meaning:** Wrong HTTP method

```http
POST /orders   (but controller expects GET)
```

---

### 6️⃣ `HttpMediaTypeNotSupportedException`

**Meaning:** Wrong `Content-Type`

```http
Content-Type: text/plain
Expected: application/json
```

---

### 7️⃣ `MissingServletRequestParameterException`

**Meaning:** Required query param missing

```java
@RequestParam("id") Long id;
```

---

### 8️⃣ `MethodArgumentTypeMismatchException`

**Meaning:** Type conversion failed

```http
?id=abc   → expected Long
```

---

### 9️⃣ `HttpMessageNotReadableException`

**Meaning:** Invalid JSON request body

**Causes**

* Malformed JSON
* Type mismatch

---

## 3. Validation Exceptions (Very Frequent)

### 🔟 `MethodArgumentNotValidException`

**Meaning:** Bean validation failed on `@RequestBody`

```java
@NotBlank
private String name;
```

**Occurs when**

* `@Valid` is used
* Input violates constraints

---

### 1️⃣1️⃣ `ConstraintViolationException`

**Meaning:** Validation failed on params / path variables

```java
@GetMapping("/{id}")
public void get(@Min(1) @PathVariable int id)
```

---

## 4. Database / JPA Exceptions (CRITICAL)

### 1️⃣2️⃣ `DataIntegrityViolationException`

**Meaning:** DB constraint violated

**Examples**

* Duplicate key
* NOT NULL violation
* Foreign key violation

---

### 1️⃣3️⃣ `JpaObjectRetrievalFailureException`

**Meaning:** Entity not found

```java
@ManyToOne(optional=false)
```

---

### 1️⃣4️⃣ `TransactionSystemException`

**Meaning:** Transaction failed

**Causes**

* Constraint violation
* Rollback failure
* Timeout

---

### 1️⃣5️⃣ `LazyInitializationException`

**Meaning:** Accessed lazy entity outside transaction

```java
Session closed
```

**Fix**

* Fetch join
* DTO projection
* Open session in service layer

---

## 5. AOP & Transaction Exceptions (INTERVIEW FAVORITES)

### 1️⃣6️⃣ `UnexpectedRollbackException`

**Meaning:** Transaction silently rolled back

**Cause**

* Inner method throws runtime exception
* Outer method unaware

---

### 1️⃣7️⃣ `IllegalTransactionStateException`

**Meaning:** Invalid transaction usage

**Example**

* Nested transactions without support

---

## 6. Security Exceptions (If Spring Security Used)

### 1️⃣8️⃣ `AccessDeniedException`

**Meaning:** Authenticated but unauthorized

---

### 1️⃣9️⃣ `AuthenticationException`

**Meaning:** Authentication failed

---

## 7. Configuration & Environment Exceptions

### 2️⃣0️⃣ `IllegalStateException`

**Meaning:** Misconfiguration

**Very common causes**

* Missing property
* Auto-configuration conflict
* Wrong profile

---

### 2️⃣1️⃣ `ConversionFailedException`

**Meaning:** Property binding failed

```yaml
timeout: abc   # expected int
```

---

## 8. Async / Threading Exceptions

### 2️⃣2️⃣ `AsyncRequestTimeoutException`

**Meaning:** Async request timed out

---

### 2️⃣3️⃣ `RejectedExecutionException`

**Meaning:** Thread pool exhausted

**Very serious in prod**

---

## 9. Common Production Mistakes That Cause Exceptions

| Mistake                   | Exception                |
| ------------------------- | ------------------------ |
| Self-invocation           | `@Transactional` ignored |
| Wrong Content-Type        | MediaType exceptions     |
| Lazy entity in controller | LazyInitialization       |
| No global handler         | 500 everywhere           |
| Circular dependencies     | Bean creation failures   |

---

## 10. MUST-HAVE Global Exception Handling

```java
@ControllerAdvice
public class GlobalExceptionHandler {

  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ResponseEntity<?> handleValidation(...) { }

  @ExceptionHandler(DataIntegrityViolationException.class)
  public ResponseEntity<?> handleDb(...) { }
}
```

---

## 11. Interview-Level Expectations

You should be able to answer:

* Difference between `MethodArgumentNotValidException` and `ConstraintViolationException`
* Why `LazyInitializationException` occurs
* Why `@Transactional` silently fails
* Why bean creation fails only in prod

---

## 12. Minimal “Must-Recognize” Exception Set

If you recognize only these, you’ll survive:

1. `BeanCreationException`
2. `NoSuchBeanDefinitionException`
3. `MethodArgumentNotValidException`
4. `HttpMessageNotReadableException`
5. `DataIntegrityViolationException`
6. `LazyInitializationException`
7. `UnexpectedRollbackException`
8. `AccessDeniedException`

---

If you want next:

* **Exception → HTTP status mapping (best practices)**
* **Tricky production exception scenarios**
* **How to design clean error responses**
* **Spring exception hierarchy explained**

Say which one.
