---
title: "Custom Annotations and Stereotypes: Meta-Annotations and @AliasFor"
description: "Part 31 of the Spring Core series. Building domain-specific custom Spring stereotype annotations using meta-annotations and @AliasFor attribute mapping."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 31

tags:
  - java
  - spring
  - spring-core
  - custom-annotations
  - stereotypes

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/custom-annotations-and-stereotypes/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 31 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 45 min / ~75 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Meta-annotations, custom Spring stereotypes, @AliasFor attribute aliasing, domain-driven annotation design |
| **Concepts unlocked** | Part 32 (Component Scanning Filters), Part 34 (Conditional Bean Registration) |

## 1. Prerequisites

**Spring concepts required**
- Component scanning and `@Component` meta-annotation rules (Part 30)

**Previous blogs required**
- Part 30 (*Classpath Scanning*)

---

## 2. What You Will Learn

- How Spring detects meta-annotated annotations as candidate stereotypes
- How to create domain-specific custom annotations (e.g., `@UseCase`, `@DatabaseAdapter`)
- How `@AliasFor` forwards custom annotation attributes to Spring's `@Component(value = ...)`
- When custom stereotypes improve architecture versus when they introduce unnecessary noise

---

## 3. Creating a Custom Stereotype Annotation

To build a custom stereotype, annotate your interface definition with `@Component`:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Meta-annotation marking this as a Spring component!
public @interface UseCase {

    @AliasFor(annotation = Component.class)
    String value() default ""; // Maps @UseCase("myBean") to @Component("myBean")
}
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog31/annotation/UseCase.java`

```java
package com.example.springcore.blog31.annotation;

import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Component;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Tells Spring classpath scanner that @UseCase is a bean!
public @interface UseCase {

    @AliasFor(annotation = Component.class)
    String value() default "";
}
```

**File:** `src/main/java/com/example/springcore/blog31/domain/PlaceOrderUseCase.java`

```java
package com.example.springcore.blog31.domain;

import com.example.springcore.blog31.annotation.UseCase;

@UseCase("placeOrderUseCase") // Custom Stereotype!
public class PlaceOrderUseCase {

    public void execute(String orderId) {
        System.out.println("[PlaceOrderUseCase (@UseCase)] Executed business logic for " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog31/CustomStereotypeDemoApp.java`

```java
package com.example.springcore.blog31;

import com.example.springcore.blog31.domain.PlaceOrderUseCase;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class CustomStereotypeDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog31.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 31 - Custom Stereotype Annotation Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            PlaceOrderUseCase useCase = context.getBean("placeOrderUseCase", PlaceOrderUseCase.class);
            useCase.execute("ORD-555");
        }
    }
}
```

Expected Output:

```text
=== Blog 31 - Custom Stereotype Annotation Demo ===
[PlaceOrderUseCase (@UseCase)] Executed business logic for ORD-555
```

---

## 5. Key Takeaways

1. Any annotation meta-annotated with `@Component` is recognized as a valid Spring stereotype.
2. `@AliasFor(annotation = Component.class)` maps custom annotation attributes to `@Component` attributes.
3. Custom stereotypes clarify domain intent in Hexagonal or Clean Architecture codebases.

---

## 6. Next Blog

**Part 32: Component Scanning Filters**
We explore `includeFilters`, `excludeFilters`, `AnnotationTypeFilter`, and custom `TypeFilter` implementations.

Where you are: **Blog 31 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
