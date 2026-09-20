---
title: "Autowiring and Collaborators: Automatic Dependency Resolution"
description: "Part 10 of the Spring Core series. How Spring autowires collaborators by type and name, resolving complex dependency graphs across XML, Java Config, and Annotations."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 10

tags:
  - java
  - spring
  - spring-core
  - autowiring
  - dependency-injection

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/autowiring-and-collaborators/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 10 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 50 min / ~90 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Autowiring by type vs by name, @Autowired semantics, implicit constructor autowiring, multi-level dependency graphs |
| **Concepts unlocked** | Part 11 (Injection style comparison), Part 12 (@Primary and @Qualifier) |

## 1. Prerequisites

**Spring concepts required**
- Beans, `ApplicationContext`, and bean lookup (Parts 1–9)

**Previous blogs required**
- Part 2 (*IoC and DI Fundamentals*)
- Part 9 (*Bean Lookup and Bean Naming*)

---

## 2. What You Will Learn

- How Spring automatically wires collaborators without explicit manual binding
- The difference between autowiring **byType** and **byName**
- How `@Autowired` resolves dependencies across single and multiple candidate beans
- How Spring resolves 3-level dependency graphs (`OrderService` ──► `PaymentService` ──► `PaymentGateway`)
- Implicit single-constructor autowiring in modern Spring (4.3+)

---

## 3. Dependency Graph Visualization

```text
OrderService (Bean)
     │
     └── @Autowired Constructor / Field
            ▼
     PaymentService (Bean)
            │
            └── @Autowired Constructor / Field
                   ▼
            PaymentGateway (Bean)
```

Spring constructs this graph bottom-up: `PaymentGateway` is instantiated first, injected into `PaymentService`, which is then injected into `OrderService`.

---

## 4. Autowiring Modes Overview

| Autowiring Strategy | Description | Config Location |
|---|---|---|
| **byType (Default)** | Matches candidate beans matching the target parameter/field type. | `@Autowired`, `@Bean` params, XML `autowire="byType"` |
| **byName** | Fallback matching candidate bean name to parameter/field name. | XML `autowire="byName"` or field name match |
| **Constructor** | Resolves constructor parameters by type. | Single constructor or `@Autowired` constructor |

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog10/domain/PaymentGateway.java`

```java
package com.example.springcore.blog10.domain;

import org.springframework.stereotype.Component;

@Component
public class PaymentGateway {
    public void charge(String orderId) {
        System.out.println("[PaymentGateway] Processing charge for " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog10/domain/PaymentService.java`

```java
package com.example.springcore.blog10.domain;

import org.springframework.stereotype.Component;

@Component
public class PaymentService {

    private final PaymentGateway gateway;

    // Implicit Autowiring (Single constructor)
    public PaymentService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public void process(String orderId) {
        gateway.charge(orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog10/domain/OrderService.java`

```java
package com.example.springcore.blog10.domain;

import org.springframework.stereotype.Component;

@Component
public class OrderService {

    private final PaymentService paymentService;

    // Implicit Autowiring (Single constructor)
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void checkout(String orderId) {
        System.out.println("[OrderService] Checkout initiated for " + orderId);
        paymentService.process(orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog10/AutowireDemoApp.java`

```java
package com.example.springcore.blog10;

import com.example.springcore.blog10.domain.OrderService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class AutowireDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog10.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 10 - Autowiring & Collaborators Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            OrderService orderService = context.getBean(OrderService.class);
            orderService.checkout("ORD-1001");
        }
    }
}
```

Expected Output:

```text
=== Blog 10 - Autowiring & Collaborators Demo ===
[OrderService] Checkout initiated for ORD-1001
[PaymentGateway] Processing charge for ORD-1001
```

---

## 6. Key Takeaways

1. Autowiring automatically connects dependent beans by inspecting parameter types.
2. Single constructors in Spring 4.3+ do not require explicit `@Autowired` annotations.
3. Dependencies are created bottom-up before dependent beans are instantiated.

---

## 7. Next Blog

**Part 11: Constructor vs Setter vs Field Injection**
A deep architectural dive comparing immutability, testability, and anti-patterns across injection styles.

Where you are: **Blog 10 of 49** (Phase 2: Dependency Resolution and Wiring).
