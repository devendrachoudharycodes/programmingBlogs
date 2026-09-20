---
title: "ObjectProvider, ObjectFactory, and Provider: Programmatic Dependency Lookup"
description: "Part 17 of the Spring Core series. How ObjectProvider, ObjectFactory, and jakarta.inject.Provider enable lazy, optional, and prototype dependency resolution."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 17

tags:
  - java
  - spring
  - spring-core
  - object-provider
  - dependency-injection

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/objectprovider-objectfactory-and-provider/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 17 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | ObjectProvider API, ObjectFactory, jakarta.inject.Provider, stream-based injection, programmatic prototype resolution |
| **Concepts unlocked** | Part 18 (Custom Bean Scopes), Part 20 (Lazy Initialization) |

## 1. Prerequisites

**Spring concepts required**
- Prototype scope and `@Lookup` method injection (Parts 15 & 16)

**Previous blogs required**
- Part 15 (*Bean Scopes*)
- Part 16 (*Method Injection and @Lookup*)

---

## 2. What You Will Learn

- How `ObjectProvider<T>` simplifies lazy, optional, and prototype bean lookups
- Methods on `ObjectProvider`: `getObject()`, `getIfAvailable()`, `getIfUnique()`, and `stream()`
- Differences between `ObjectProvider<T>`, `ObjectFactory<T>`, and `jakarta.inject.Provider<T>`
- Why `ObjectProvider` is the modern idiomatic replacement for `@Lookup` in Spring 5+

---

## 3. Comparison of Programmatic Providers

| API | Origin | Key Capabilities |
|---|---|---|
| **`ObjectFactory<T>`** | Spring 1.0 | Basic `.getObject()` resolution. |
| **`ObjectProvider<T>`** | Spring 4.3+ | `getIfAvailable()`, `getIfUnique()`, `stream()`, `orderedStream()`. **Recommended**. |
| **`Provider<T>`** | JSR-330 (`jakarta.inject`) | Standard Java `.get()` contract for portable code. |

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog17/domain/PrototypeReport.java`

```java
package com.example.springcore.blog17.domain;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototype")
public class PrototypeReport {
    public void generate(String title) {
        System.out.println("[Report] Generating '" + title + "' (Instance ID: " + System.identityHashCode(this) + ")");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog17/domain/ReportService.java`

```java
package com.example.springcore.blog17.domain;

import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Component;

@Component
public class ReportService {

    private final ObjectProvider<PrototypeReport> reportProvider;

    // Inject ObjectProvider instead of PrototypeReport directly
    public ReportService(ObjectProvider<PrototypeReport> reportProvider) {
        this.reportProvider = reportProvider;
    }

    public void createWeeklyReports() {
        // Fetch fresh prototype instance on demand
        PrototypeReport r1 = reportProvider.getObject();
        r1.generate("Sales Summary");

        PrototypeReport r2 = reportProvider.getObject();
        r2.generate("Inventory Status");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog17/ProviderDemoApp.java`

```java
package com.example.springcore.blog17;

import com.example.springcore.blog17.domain.ReportService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class ProviderDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog17.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 17 - ObjectProvider Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            ReportService service = context.getBean(ReportService.class);
            service.createWeeklyReports();
        }
    }
}
```

Expected Output:

```text
=== Blog 17 - ObjectProvider Demo ===
[Report] Generating 'Sales Summary' (Instance ID: 198231024)
[Report] Generating 'Inventory Status' (Instance ID: 882910391)
```

---

## 5. Key Takeaways

1. `ObjectProvider<T>` allows retrieving `prototype` beans dynamically inside `singleton` beans without CGLIB abstract method overrides.
2. Provides null-safe lookup methods: `getIfAvailable(Supplier)` and `getIfUnique()`.
3. Supports stream operations (`stream()`) over all matching beans in the container.

---

## 6. Next Blog

**Part 18: Custom Bean Scopes**
We implement a custom Spring bean scope from scratch by implementing the `Scope` interface.

Where you are: **Blog 17 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
