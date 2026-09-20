---
title: "Lazy Initialization and @DependsOn: Controlling Instantiation Timing"
description: "Part 20 of the Spring Core series. Deferring bean instantiation with @Lazy and explicitly controlling creation order using @DependsOn."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 20

tags:
  - java
  - spring
  - spring-core
  - lazy-initialization
  - depends-on

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/lazy-initialization-and-dependson/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 20 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 25 min / 30 min / ~55 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Eager vs lazy singleton instantiation, @Lazy on beans & injection points, @DependsOn explicit creation order |
| **Concepts unlocked** | Part 21 (Lifecycle and SmartLifecycle), Part 45 (Refresh Process) |

## 1. Prerequisites

**Spring concepts required**
- Singleton pre-instantiation and bean lifecycle (Parts 3 & 19)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 19 (*Complete Bean Lifecycle*)

---

## 2. What You Will Learn

- Why Spring pre-instantiates singleton beans eagerly by default
- How `@Lazy` defers bean instantiation until first explicit lookup or usage
- Using `@Lazy` on injection points to inject lazy proxy placeholders
- Controlling implicit creation order across unrelated beans with `@DependsOn`
- Trade-offs between fast startup times and delayed error reporting

---

## 3. `@Lazy` vs `@DependsOn` Concepts

### `@Lazy`
By default, `ApplicationContext` pre-instantiates singletons during startup (`refresh()`). Marking a bean `@Lazy` defers instantiation until the bean is requested via `getBean()` or injected into another active bean.

### `@DependsOn`
Forces Spring to instantiate specified dependency beans **before** the annotated bean, even when no direct Java reference exists between them (e.g. database schema driver initializing before cache service).

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog20/domain/DatabaseInitializer.java`

```java
package com.example.springcore.blog20.domain;

import org.springframework.stereotype.Component;

@Component("dbInitializer")
public class DatabaseInitializer {
    public DatabaseInitializer() {
        System.out.println("[DatabaseInitializer] Initializing database tables...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog20/domain/CacheService.java`

```java
package com.example.springcore.blog20.domain;

import org.springframework.context.annotation.DependsOn;
import org.springframework.stereotype.Component;

@Component
@DependsOn("dbInitializer") // Ensures dbInitializer runs BEFORE CacheService!
public class CacheService {
    public CacheService() {
        System.out.println("[CacheService] Cache initialized (depends on dbInitializer).");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog20/domain/HeavyReportEngine.java`

```java
package com.example.springcore.blog20.domain;

import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Component;

@Component
@Lazy // Deferred instantiation!
public class HeavyReportEngine {
    public HeavyReportEngine() {
        System.out.println("[HeavyReportEngine] LAZY Instantiation executed!");
    }

    public void runReport() {
        System.out.println("[HeavyReportEngine] Running report...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog20/LazyDemoApp.java`

```java
package com.example.springcore.blog20;

import com.example.springcore.blog20.domain.HeavyReportEngine;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class LazyDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog20.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 20 - Lazy & @DependsOn Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("Context startup complete! HeavyReportEngine is lazy and has NOT been built yet.");

            System.out.println("\nRequesting HeavyReportEngine bean...");
            HeavyReportEngine engine = context.getBean(HeavyReportEngine.class);
            engine.runReport();
        }
    }
}
```

Expected Output:

```text
=== Blog 20 - Lazy & @DependsOn Demo ===
[DatabaseInitializer] Initializing database tables...
[CacheService] Cache initialized (depends on dbInitializer).
Context startup complete! HeavyReportEngine is lazy and has NOT been built yet.

Requesting HeavyReportEngine bean...
[HeavyReportEngine] LAZY Instantiation executed!
[HeavyReportEngine] Running report...
```

---

## 5. Key Takeaways

1. Eager instantiation (default) catches configuration errors at startup.
2. `@Lazy` delays instantiation until first usage, reducing startup memory and time.
3. `@DependsOn` enforces explicit creation order for beans that lack direct Java constructor references.

---

## 6. Next Blog

**Part 21: `Lifecycle`, `SmartLifecycle` and Graceful Shutdown**
We explore container start/stop signals, startup phase ordering, and graceful shutdown of background tasks.

Where you are: **Blog 20 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
