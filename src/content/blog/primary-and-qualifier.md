---
title: "@Primary and @Qualifier: Resolving Dependency Ambiguity"
description: "Part 12 of the Spring Core series. How to resolve NoUniqueBeanDefinitionException when multiple beans implement an interface using @Primary and @Qualifier."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 12

tags:
  - java
  - spring
  - spring-core
  - primary
  - qualifier

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/primary-and-qualifier/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 12 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 35 min / 45 min / ~80 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | NoUniqueBeanDefinitionException, @Primary preference, @Qualifier explicit selection, custom qualifiers, name fallback |
| **Concepts unlocked** | Part 13 (Collection, Generic and Optional Injection), Part 47 (Dependency Resolution Internals) |

## 1. Prerequisites

**Spring concepts required**
- Autowiring by type and interface-based DI (Parts 2 & 10)

**Previous blogs required**
- Part 10 (*Autowiring and Collaborators*)
- Part 11 (*Constructor vs Setter vs Field Injection*)

---

## 2. What You Will Learn

- How `NoUniqueBeanDefinitionException` occurs when multiple beans implement an interface
- How `@Primary` designates a default candidate among multiple beans of the same type
- How `@Qualifier` provides fine-grained explicit selection at injection points
- How Spring resolves candidate selection precedence
- Fallback matching by parameter or field name

---

## 3. The Ambiguity Problem

When an interface `PaymentGateway` has 3 implementations (`StripePaymentGateway`, `RazorpayPaymentGateway`, `PayPalPaymentGateway`) registered in the container, Spring's `byType` autowiring fails with:

`NoUniqueBeanDefinitionException: No qualifying bean of type 'PaymentGateway' available: expected single matching bean but found 3: stripePaymentGateway, razorpayPaymentGateway, payPalPaymentGateway`

---

## 4. Disambiguation Strategies

### Strategy 1: `@Primary`
Marks one bean as the default fallback when multiple candidates exist.

```java
@Component
@Primary
public class StripePaymentGateway implements PaymentGateway { ... }
```

### Strategy 2: `@Qualifier`
Overrides `@Primary` and explicitly targets a specific bean by name at the injection site.

```java
@Component
public class OrderService {
    private final PaymentGateway gateway;

    public OrderService(@Qualifier("razorpayPaymentGateway") PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

---

## 5. Resolution Precedence Rules

```text
Injection Point Request
       │
       ▼
Is @Qualifier present? ──YES──► Match by Qualifier Name
       │
      NO
       ▼
Is one bean marked @Primary? ──YES──► Select @Primary Bean
       │
      NO
       ▼
Does parameter/field name match a bean name? ──YES──► Select Matching Bean Name
       │
      NO
       ▼
Throw NoUniqueBeanDefinitionException!
```

---

## 6. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog12/domain/PaymentGateway.java`

```java
package com.example.springcore.blog12.domain;

public interface PaymentGateway {
    void charge();
}
```

**File:** `src/main/java/com/example/springcore/blog12/domain/StripeGateway.java`

```java
package com.example.springcore.blog12.domain;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

@Component
@Primary // Default gateway when no qualifier is specified
public class StripeGateway implements PaymentGateway {
    @Override
    public void charge() {
        System.out.println("[StripeGateway (Primary)] Charging payment...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog12/domain/RazorpayGateway.java`

```java
package com.example.springcore.blog12.domain;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
@Qualifier("razorpay")
public class RazorpayGateway implements PaymentGateway {
    @Override
    public void charge() {
        System.out.println("[RazorpayGateway (Qualifier)] Charging payment...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog12/domain/CheckoutService.java`

```java
package com.example.springcore.blog12.domain;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
public class CheckoutService {

    private final PaymentGateway defaultGateway;
    private final PaymentGateway razorpayGateway;

    public CheckoutService(
            PaymentGateway defaultGateway, // Injects Stripe (@Primary)
            @Qualifier("razorpay") PaymentGateway razorpayGateway) { // Injects Razorpay
        this.defaultGateway = defaultGateway;
        this.razorpayGateway = razorpayGateway;
    }

    public void run() {
        System.out.print("Default Checkout: ");
        defaultGateway.charge();

        System.out.print("Explicit Qualifier Checkout: ");
        razorpayGateway.charge();
    }
}
```

**File:** `src/main/java/com/example/springcore/blog12/AmbiguityDemoApp.java`

```java
package com.example.springcore.blog12;

import com.example.springcore.blog12.domain.CheckoutService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class AmbiguityDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog12.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 12 - @Primary & @Qualifier Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            CheckoutService service = context.getBean(CheckoutService.class);
            service.run();
        }
    }
}
```

Expected Output:

```text
=== Blog 12 - @Primary & @Qualifier Demo ===
Default Checkout: [StripeGateway (Primary)] Charging payment...
Explicit Qualifier Checkout: [RazorpayGateway (Qualifier)] Charging payment...
```

---

## 7. Key Takeaways

1. `@Primary` defines a single default bean when multiple beans of a type exist.
2. `@Qualifier` explicitly targets a specific bean, taking precedence over `@Primary`.
3. If neither `@Primary` nor `@Qualifier` match, Spring falls back to matching parameter/field name against registered bean names.

---

## 8. Next Blog

**Part 13: Collection, Generic, and Optional Injection; `@Order` and `@Priority`**
We explore injecting `List<T>`, `Set<T>`, `Map<String, T>`, generic interfaces, and ordering dependencies.

Where you are: **Blog 12 of 49** (Phase 2: Dependency Resolution and Wiring).
