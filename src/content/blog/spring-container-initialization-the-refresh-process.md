---
title: "Spring Container Initialization: The 13 Phases of AbstractApplicationContext.refresh()"
description: "Part 45 of the Spring Core series. Complete architectural deep dive into AbstractApplicationContext.refresh() and all 13 container initialization phases."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 45

tags:
  - java
  - spring
  - spring-core
  - refresh-process
  - container-internals

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/spring-container-initialization-the-refresh-process/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 45 of 49 |
| **Difficulty** | ⚫ Internals / Architecture |
| **Reading / Coding / Total** | 45 min / 45 min / ~90 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | AbstractApplicationContext.refresh() template method, all 13 refresh phases, eager singleton instantiation timing |
| **Concepts unlocked** | Part 46 (Singleton Registry and Three-Level Cache), Part 47 (Dependency Resolution Internals) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` internal delegation, post-processors, and event multicaster (Parts 22, 23 & 42)

**Previous blogs required**
- Part 23 (*Container Extension Points*)
- Part 42 (*ApplicationContext Internal Architecture*)

---

## 2. What You Will Learn

- Why `refresh()` is the master template method that bootstraps the entire Spring container
- The exact sequence of all 13 phases executed inside `AbstractApplicationContext.refresh()`
- When `BeanFactoryPostProcessor`s and `BeanPostProcessor`s are invoked during refresh
- Exactly when non-lazy singleton beans are instantiated (`finishBeanFactoryInitialization()`)

---

## 3. The 13 Phases of `refresh()`

```text
 1. prepareRefresh()                  ── Validates environment & active profiles
 2. obtainFreshBeanFactory()          ── Creates or loads underlying DefaultListableBeanFactory
 3. prepareBeanFactory()              ── Registers standard context classloaders & Aware processors
 4. postProcessBeanFactory()          ── Template hook for subclass post-processing
 5. invokeBeanFactoryPostProcessors() ── Invokes BeanDefinitionRegistryPostProcessor & BFPPs
 6. registerBeanPostProcessors()       ── Instantiates & registers BeanPostProcessor beans
 7. initMessageSource()               ── Initializes MessageSource for i18n
 8. initApplicationEventMulticaster()── Initializes SimpleApplicationEventMulticaster
 9. onRefresh()                       ── Template hook for web/theme initialization
10. registerListeners()               ── Registers ApplicationListener beans
11. finishBeanFactoryInitialization() ── Pre-instantiates all non-lazy SINGLETON beans!
12. finishRefresh()                   ── Publishes ContextRefreshedEvent & starts Lifecycle beans
13. resetCommonCaches()               ── Clears temporary reflection caches
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog45/RefreshPhasesDemoApp.java`

```java
package com.example.springcore.blog45;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

public class RefreshPhasesDemoApp {

    @Configuration
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 45 - Container Refresh Process Demo ===");

        // AnnotationConfigApplicationContext constructor invokes refresh() internally!
        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("Container refresh() completed successfully!");
            System.out.println("Container startup timestamp: " + context.getStartupDate());
        }
    }
}
```

Expected Output:

```text
=== Blog 45 - Container Refresh Process Demo ===
Container refresh() completed successfully!
Container startup timestamp: 1774218800000
```

---

## 5. Key Takeaways

1. `AbstractApplicationContext.refresh()` is the single master initialization routine.
2. Metadata modifications happen in Phase 5 (`invokeBeanFactoryPostProcessors`).
3. Singletons are instantiated in Phase 11 (`finishBeanFactoryInitialization`).

---

## 6. Next Blog

**Part 46: Singleton Registry and the Three-Level Cache**
We analyze `DefaultSingletonBeanRegistry` internals, `singletonObjects`, `earlySingletonObjects`, and `singletonFactories`.

Where you are: **Blog 45 of 49** (Phase 5: Architecture, Internals, and Diagnostics).
