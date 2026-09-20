---
title: "ApplicationContext Implementations: Standalone, XML, Annotation, and Generic"
description: "Part 43 of the Spring Core series. Comparing AnnotationConfigApplicationContext, ClassPathXmlApplicationContext, FileSystemXmlApplicationContext, and GenericApplicationContext."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 43

tags:
  - java
  - spring
  - spring-core
  - application-context
  - implementations

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/applicationcontext-implementations/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 43 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | ApplicationContext implementation choices, AnnotationConfigApplicationContext, ClassPathXmlApplicationContext, FileSystemXmlApplicationContext, GenericApplicationContext |
| **Concepts unlocked** | Part 44 (ApplicationContext Hierarchy), Part 45 (Refresh Process) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` concepts and configuration styles (Parts 4 & 42)

**Previous blogs required**
- Part 4 (*Building Spring from Scratch with Maven*)
- Part 42 (*ApplicationContext Internal Architecture*)

---

## 2. Implementation Overview Matrix

| Implementation Class | Configuration Format | Typical Use Case |
|---|---|---|
| **`AnnotationConfigApplicationContext`** | Java Config & Component Annotations | Standard for modern standalone Spring applications. |
| **`ClassPathXmlApplicationContext`** | XML files on Classpath | Legacy applications with `beans.xml` in classpath. |
| **`FileSystemXmlApplicationContext`** | XML files on File System | Applications loading XML from absolute file paths. |
| **`GenericApplicationContext`** | Programmatic Bean Registrations | Functional bean registration and micro-benchmarks. |

---

## 3. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog43/ContextImplementationsDemoApp.java`

```java
package com.example.springcore.blog43;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.GenericApplicationContext;

public class ContextImplementationsDemoApp {

    @Configuration
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 43 - ApplicationContext Implementations Demo ===");

        // 1. AnnotationConfigApplicationContext
        try (AnnotationConfigApplicationContext ctx1 =
                     new AnnotationConfigApplicationContext(Config.class)) {
            System.out.println("1. AnnotationConfigApplicationContext active: " + ctx1.isActive());
        }

        // 2. GenericApplicationContext (Functional)
        try (GenericApplicationContext ctx2 = new GenericApplicationContext()) {
            ctx2.registerBean("myString", String.class, () -> "GenericContextBean");
            ctx2.refresh();
            System.out.println("2. GenericApplicationContext bean: " + ctx2.getBean("myString"));
        }
    }
}
```

Expected Output:

```text
=== Blog 43 - ApplicationContext Implementations Demo ===
1. AnnotationConfigApplicationContext active: true
2. GenericApplicationContext bean: GenericContextBean
```

---

## 4. Key Takeaways

1. `AnnotationConfigApplicationContext` is the standard choice for Java/Annotation configurations.
2. `GenericApplicationContext` is used for functional bean registration without reflection scanning.
3. XML-based contexts (`ClassPathXmlApplicationContext`) exist primarily for backward compatibility.

---

## 5. Next Blog

**Part 44: `ApplicationContext` Hierarchy**
We explore parent-child context hierarchies, bean inheritance, and isolation boundaries.

Where you are: **Blog 43 of 49** (Phase 5: Architecture, Internals, and Diagnostics).
