---
title: "Constructor vs Setter vs Field Injection: Deep Architectural Comparison"
description: "Part 11 of the Spring Core series. Comparing constructor, setter, and field injection across immutability, testability, null-safety, and circular dependency handling."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 11

tags:
  - java
  - spring
  - spring-core
  - constructor-injection
  - field-injection

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/constructor-vs-setter-vs-field-injection/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 11 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 40 min / ~75 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Immutability, null safety, unit testing without container, circular dependency impact, field injection anti-pattern |
| **Concepts unlocked** | Part 12 (@Primary and @Qualifier), Part 14 (Circular Dependencies) |

## 1. Prerequisites

**Spring concepts required**
- Autowiring and dependency injection mechanisms (Parts 2 & 10)

**Previous blogs required**
- Part 2 (*IoC and DI Fundamentals*)
- Part 10 (*Autowiring and Collaborators*)

---

## 2. What You Will Learn

- Architectural comparison across Immutability, Testability, and Null-Safety
- Why **Constructor Injection** is the industry standard for required dependencies
- When **Setter Injection** is appropriate (optional or reconfigurable dependencies)
- Why **Field Injection** is an anti-pattern in production code
- How to write plain JUnit tests for constructor-injected classes without starting Spring

---

## 3. Comparison Matrix

| Evaluation Criteria | Constructor Injection | Setter Injection | Field Injection |
|---|---|---|---|
| **Field Immutability (`final`)** | ✅ Supported (`private final`) | ❌ Impossible | ❌ Impossible |
| **Null Safety & Valid State** | ✅ Guaranteed at instantiation | ❌ Possible partially-initialized object | ❌ Field is null until injected |
| **Plain Unit Testability** | ✅ Trivial (`new Service(mock)`) | 🟡 Requires setter calls | ❌ Requires Reflection / Spring Context |
| **Dependency Smell Visibility** | ✅ Obvious if constructor has 10 params | 🟡 Hidden across setters | ❌ Completely hidden |
| **Optional Dependencies** | 🟡 Represented via `Optional<T>` | ✅ Natural (setter optionally called) | 🟡 Supported via `required=false` |
| **Circular Dependencies** | ❌ Fails fast (`BeanCurrentlyInCreationException`) | 🟡 Resolved automatically | 🟡 Resolved automatically |

---

## 4. Demonstration: Plain Unit Testing Without Spring

**File:** `src/main/java/com/example/springcore/blog11/domain/OrderService.java`

```java
package com.example.springcore.blog11.domain;

public class OrderService {

    private final PaymentService paymentService; // Immutable field

    public OrderService(PaymentService paymentService) {
        if (paymentService == null) {
            throw new IllegalArgumentException("PaymentService cannot be null!");
        }
        this.paymentService = paymentService;
    }

    public boolean processOrder(String orderId) {
        return paymentService.pay(orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog11/domain/PaymentService.java`

```java
package com.example.springcore.blog11.domain;

public interface PaymentService {
    boolean pay(String orderId);
}
```

**File:** `src/main/java/com/example/springcore/blog11/UnitTestDemoApp.java`

```java
package com.example.springcore.blog11;

import com.example.springcore.blog11.domain.OrderService;
import com.example.springcore.blog11.domain.PaymentService;

public class UnitTestDemoApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 11 - Plain Unit Test Without Spring Context ===");

        // Mock collaborator created with a lambda (no Spring container needed!)
        PaymentService mockPaymentService = orderId -> {
            System.out.println("[Test Mock] Mock payment approved for " + orderId);
            return true;
        };

        // Instant unit test execution
        OrderService orderService = new OrderService(mockPaymentService);
        boolean result = orderService.processOrder("TEST-999");

        System.out.println("Test Execution Result: " + (result ? "PASSED" : "FAILED"));
    }
}
```

Expected Output:

```text
=== Blog 11 - Plain Unit Test Without Spring Context ===
[Test Mock] Mock payment approved for TEST-999
Test Execution Result: PASSED
```

---

## 5. Key Takeaways

1. Use **Constructor Injection** for all required dependencies (`private final`).
2. Use **Setter Injection** sparingly for optional or fallback dependencies.
3. Avoid **Field Injection** (`@Autowired private Service s;`) to maintain immutability and unit testability.

---

## 6. Next Blog

**Part 12: `@Primary` and `@Qualifier`**
We solve ambiguity errors when multiple candidate beans implement the same interface.

Where you are: **Blog 11 of 49** (Phase 2: Dependency Resolution and Wiring).
