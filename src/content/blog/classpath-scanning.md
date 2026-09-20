---
title: "Classpath Scanning: Automatic Component Discovery"
description: "Part 30 of the Spring Core series. How @ComponentScan discovers candidate components, ASM byte-code inspection, and stereotype annotation mapping."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 30

tags:
  - java
  - spring
  - spring-core
  - component-scan
  - stereotypes

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/classpath-scanning/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 30 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 40 min / 45 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | @ComponentScan, ClasspathBeanDefinitionScanner, ASM bytecode metadata reading, candidate component detection |
| **Concepts unlocked** | Part 31 (Custom Annotations and Stereotypes), Part 32 (Component Scanning Filters) |

## 1. Prerequisites

**Spring concepts required**
- Component annotations and `BeanDefinition` recipes (Parts 10 & 26)

**Previous blogs required**
- Part 26 (*BeanDefinition*)
- Part 28 (*Annotation Processing Infrastructure*)

---

## 2. What You Will Learn

- How `@ComponentScan` automatically detects `@Component`, `@Service`, `@Repository`, and `@Controller` classes
- Why Spring uses **ASM bytecode inspection** (`SimpleMetadataReader`) to read class annotations without loading classes into the JVM
- Base package resolution strategies: String package names vs type-safe `basePackageClasses`
- XML `<context:component-scan>` equivalent configuration

---

## 3. How Component Scanning Works Internally

```text
@ComponentScan(basePackages = "com.example")
       │
       ▼
ClasspathBeanDefinitionScanner (Walks classpath .class files)
       │
       ▼
SimpleMetadataReader (Reads bytecode via ASM without ClassLoader.loadClass)
       │
       ▼
Is class annotated with @Component or meta-annotated stereotype?
       ├── YES ──► Create ScannedGenericBeanDefinition ──► Register in BeanDefinitionRegistry
       └── NO  ──► Skip class
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog30/domain/OrderRepository.java`

```java
package com.example.springcore.blog30.domain;

import org.springframework.stereotype.Repository;

@Repository
public class OrderRepository {
    public void save() {
        System.out.println("[OrderRepository (@Repository)] Saved order to database!");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog30/domain/OrderService.java`

```java
package com.example.springcore.blog30.domain;

import org.springframework.stereotype.Service;

@Service
public class OrderService {
    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }

    public void process() {
        System.out.println("[OrderService (@Service)] Processing order...");
        repository.save();
    }
}
```

**File:** `src/main/java/com/example/springcore/blog30/ScanningDemoApp.java`

```java
package com.example.springcore.blog30;

import com.example.springcore.blog30.domain.OrderService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class ScanningDemoApp {

    @Configuration
    // Type-safe scanning using basePackageClasses prevents package rename bugs!
    @ComponentScan(basePackageClasses = OrderService.class)
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 30 - Classpath Scanning Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            OrderService service = context.getBean(OrderService.class);
            service.process();
        }
    }
}
```

Expected Output:

```text
=== Blog 30 - Classpath Scanning Demo ===
[OrderService (@Service)] Processing order...
[OrderRepository (@Repository)] Saved order to database!
```

---

## 5. Key Takeaways

1. `@ComponentScan` uses ASM bytecode inspection to detect candidate components efficiently without loading classes.
2. Standard stereotypes: `@Component` (generic), `@Service` (business logic), `@Repository` (persistence with exception translation), `@Controller` (web handlers).
3. Use `basePackageClasses = MyConfig.class` for refactor-friendly, type-safe scanning.

---

## 6. Next Blog

**Part 31: Custom Annotations and Stereotypes**
We build custom Spring stereotype annotations using meta-annotations and `@AliasFor`.

Where you are: **Blog 30 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
