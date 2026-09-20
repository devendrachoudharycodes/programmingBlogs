---
title: "Bean Lookup and Bean Naming: Identifiers, Aliases, and Search Strategies"
description: "Part 9 of the Spring Core series. How Spring names beans, explicit vs generated names, aliases, and programmatic lookup with getBean by name and type."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 9

tags:
  - java
  - spring
  - spring-core
  - bean-naming
  - bean-lookup

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/bean-lookup-and-bean-naming/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 9 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 35 min / ~65 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Default vs explicit bean names, aliases, getBean(Class), getBean(String, Class), BeanNameGenerator |
| **Concepts unlocked** | Part 10 (Autowiring), Part 12 (@Primary and @Qualifier) |

## 1. Prerequisites

**Spring concepts required**
- Beans, `BeanDefinition`, and IoC container basics (Parts 1–8)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 8 (*Bean Creation and Initialization*)

---

## 2. What You Will Learn

- How Spring assigns default bean names across XML, Java Config, and Annotations
- The difference between Java variable names, class names, and Spring bean identifiers
- How to define explicit bean names and aliases
- Programmatic lookup methods: `getBean(Class)`, `getBean(String)`, and `getBean(String, Class)`
- How to handle `NoUniqueBeanDefinitionException` and `NoSuchBeanDefinitionException`

---

## 3. Bean Naming Rules Across Configuration Styles

### 1. Annotation Component Scanning (`@Component`)
- **Default Name**: Class name with uncapitalized first letter (`StripePaymentGateway` ──► `stripePaymentGateway`).
- **Explicit Name**: `@Component("customGateway")`

### 2. Java Configuration (`@Bean`)
- **Default Name**: `@Bean` method name (`public PaymentService paymentService()` ──► `paymentService`).
- **Explicit Name**: `@Bean(name = {"primaryGateway", "defaultGateway"})`

### 3. XML Configuration (`<bean>`)
- **Explicit Identifier**: `<bean id="paymentGateway" class="...">`
- **Aliases**: `<alias name="paymentGateway" alias="legacyGateway"/>`

---

## 4. Programmatic Lookup Overloads

```java
// 1. Lookup by Type (Fails with NoUniqueBeanDefinitionException if multiple beans exist)
PaymentGateway g1 = context.getBean(PaymentGateway.class);

// 2. Lookup by Name (Requires explicit casting)
Object g2 = context.getBean("stripePaymentGateway");

// 3. Lookup by Name AND Type (Type-safe & Ambiguity-safe)
PaymentGateway g3 = context.getBean("stripePaymentGateway", PaymentGateway.class);
```

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog09/domain/PaymentGateway.java`

```java
package com.example.springcore.blog09.domain;

public interface PaymentGateway {
    void process();
}
```

**File:** `src/main/java/com/example/springcore/blog09/domain/StripeGateway.java`

```java
package com.example.springcore.blog09.domain;

import org.springframework.stereotype.Component;

@Component("stripeGateway")
public class StripeGateway implements PaymentGateway {
    @Override
    public void process() {
        System.out.println("[StripeGateway] Processing payment...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog09/domain/PayPalGateway.java`

```java
package com.example.springcore.blog09.domain;

import org.springframework.stereotype.Component;

@Component("paypalGateway")
public class PayPalGateway implements PaymentGateway {
    @Override
    public void process() {
        System.out.println("[PayPalGateway] Processing payment...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog09/NamingDemoApp.java`

```java
package com.example.springcore.blog09;

import com.example.springcore.blog09.domain.PaymentGateway;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;

public class NamingDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog09.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 09 - Bean Naming and Lookup Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            // Print all registered application bean names
            String[] names = Arrays.stream(context.getBeanDefinitionNames())
                    .filter(n -> !n.contains("."))
                    .toArray(String[]::new);

            System.out.println("Registered Beans: " + Arrays.toString(names));

            // Lookup by explicit name and type
            PaymentGateway stripe = context.getBean("stripeGateway", PaymentGateway.class);
            stripe.process();

            PaymentGateway paypal = context.getBean("paypalGateway", PaymentGateway.class);
            paypal.process();
        }
    }
}
```

Expected Output:

```text
=== Blog 09 - Bean Naming and Lookup Demo ===
Registered Beans: [paypalGateway, stripeGateway]
[StripeGateway] Processing payment...
[PayPalGateway] Processing payment...
```

---

## 6. Key Takeaways

1. Default `@Component` bean names uncapitalize the class name (`StripeGateway` ──► `stripeGateway`).
2. Default `@Bean` method names use the method identifier.
3. Always prefer `getBean("beanName", TargetClass.class)` over untyped `getBean("beanName")`.

---

## 7. Next Blog

**Part 10: Autowiring and Collaborators**
We dive into autowiring strategies, collaborators, and automatic dependency resolution algorithms.

Where you are: **Blog 9 of 49** (Phase 2: Dependency Resolution and Wiring).
