---
title: "SpEL Fundamentals: Spring Expression Language Syntax and Evaluation"
description: "Part 38 of the Spring Core series. Deep dive into Spring Expression Language (SpEL) syntax, bean references, safe navigation (?.), and evaluation contexts."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 38

tags:
  - java
  - spring
  - spring-core
  - spel
  - expression-language

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/spel-fundamentals/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 38 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 45 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | SpEL syntax, @Value("#{...}"), bean reference (@beanName), safe navigation (?.), Elvis operator (?:), SpelExpressionParser |
| **Concepts unlocked** | Part 40 (Application Events), Part 47 (Dependency Resolution Internals) |

## 1. Prerequisites

**Spring concepts required**
- Property placeholders and `@Value` (Part 36)

**Previous blogs required**
- Part 36 (*@Value and Property Resolution*)

---

## 2. What You Will Learn

- How SpEL expressions `#{...}` differ from property placeholders `${...}`
- Accessing bean methods and properties directly in expressions: `#{@beanName.method()}`
- Safe navigation operator `?.` and Elvis operator `?:`
- Evaluating expressions programmatically with `SpelExpressionParser` and `EvaluationContext`

---

## 3. SpEL Operator Quick Reference

| Operator Syntax | Description | Example |
|---|---|---|
| **Bean Reference** | `@beanName` | `@Value("#{@taxCalculator.calculateRate()}")` |
| **Elvis Operator** | `a ?: b` (Default if null) | `@Value("#{systemProperties['custom.env'] ?: 'DEFAULT_ENV'}")` |
| **Safe Navigation** | `a?.b` (Null-safe property) | `@Value("#{order?.customer?.email}")` |
| **Relational / Logical** | `and`, `or`, `eq`, `gt` | `@Value("#{systemProperties['user.country'] eq 'US'}")` |

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog38/domain/TaxCalculator.java`

```java
package com.example.springcore.blog38.domain;

import org.springframework.stereotype.Component;

@Component("taxCalculator")
public class TaxCalculator {
    public double getTaxRate() {
        return 0.18; // 18% tax rate
    }
}
```

**File:** `src/main/java/com/example/springcore/blog38/domain/InvoiceService.java`

```java
package com.example.springcore.blog38.domain;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class InvoiceService {

    private final double computedTaxRate;
    private final String activeOS;

    public InvoiceService(
            // Evaluates method on 'taxCalculator' bean via SpEL!
            @Value("#{@taxCalculator.getTaxRate()}") double computedTaxRate,
            // Accesses system property via SpEL
            @Value("#{systemProperties['os.name']}") String activeOS) {
        this.computedTaxRate = computedTaxRate;
        this.activeOS = activeOS;
    }

    public void printInvoiceDetails() {
        System.out.println("[InvoiceService] Computed Tax Rate : " + (computedTaxRate * 100) + "%");
        System.out.println("[InvoiceService] Active OS          : " + activeOS);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog38/SpelDemoApp.java`

```java
package com.example.springcore.blog38;

import com.example.springcore.blog38.domain.InvoiceService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class SpelDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog38.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 38 - SpEL Fundamentals Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            InvoiceService service = context.getBean(InvoiceService.class);
            service.printInvoiceDetails();
        }
    }
}
```

Expected Output:

```text
=== Blog 38 - SpEL Fundamentals Demo ===
[InvoiceService] Computed Tax Rate : 18.0%
[InvoiceService] Active OS          : Linux
```

---

## 5. Key Takeaways

1. `${...}` resolves properties from `Environment`; `#{...}` evaluates SpEL expressions at runtime.
2. Use `@beanName` inside SpEL expressions to invoke methods or read properties from other beans.
3. Use safe navigation `?.` to prevent `NullPointerException`s during nested property traversal.

---

## 6. Next Blog

**Part 39: Resource Abstraction**
We explore Spring's `Resource` and `ResourceLoader` abstractions for unified file and classpath IO.

Where you are: **Blog 38 of 49** (Phase 4b: Environment and Infrastructure Abstractions).
