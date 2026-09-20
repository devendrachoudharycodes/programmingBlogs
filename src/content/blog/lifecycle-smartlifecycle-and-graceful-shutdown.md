---
title: "Lifecycle, SmartLifecycle, and Graceful Shutdown"
description: "Part 21 of the Spring Core series. How SmartLifecycle controls background component startup, phased execution, and graceful shutdown signal management."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 21

tags:
  - java
  - spring
  - spring-core
  - smart-lifecycle
  - graceful-shutdown

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/lifecycle-smartlifecycle-and-graceful-shutdown/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 21 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Lifecycle vs SmartLifecycle, start/stop context signals, phase ordering, graceful shutdown callbacks |
| **Concepts unlocked** | Part 22 (BeanPostProcessor), Part 40 (Application Events) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` lifecycle, bean creation, and destruction (Parts 6 & 19)

**Previous blogs required**
- Part 6 (*BeanFactory vs ApplicationContext*)
- Part 19 (*Complete Bean Lifecycle*)

---

## 2. What You Will Learn

- Difference between `Lifecycle` and `SmartLifecycle`
- How `SmartLifecycle.isAutoStartup()` automatically triggers component startup on context refresh
- Controlling startup/shutdown order using `getPhase()`
- Implementing graceful async shutdown using `stop(Runnable callback)`
- Registering JVM shutdown hooks with `registerShutdownHook()`

---

## 3. `Lifecycle` vs `SmartLifecycle`

- **`Lifecycle`**: Basic contract (`start()`, `stop()`, `isRunning()`). Requires **explicit manual invocation** of `context.start()` and `context.stop()`.
- **`SmartLifecycle`**: Extends `Lifecycle`. Adds auto-startup (`isAutoStartup()`), phased ordering (`getPhase()`), and non-blocking asynchronous stop callbacks (`stop(Runnable callback)`).

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog21/domain/BackgroundWorker.java`

```java
package com.example.springcore.blog21.domain;

import org.springframework.context.SmartLifecycle;
import org.springframework.stereotype.Component;

@Component
public class BackgroundWorker implements SmartLifecycle {

    private boolean running = false;

    @Override
    public boolean isAutoStartup() {
        return true; // Auto-start on ApplicationContext refresh
    }

    @Override
    public int getPhase() {
        return 100; // Phase ordering (lower numbers start first, stop last)
    }

    @Override
    public void start() {
        System.out.println("[BackgroundWorker] Automatic start signal received! Starting worker thread...");
        this.running = true;
    }

    @Override
    public void stop() {
        System.out.println("[BackgroundWorker] Stop signal received!");
        this.running = false;
    }

    @Override
    public void stop(Runnable callback) {
        stop();
        System.out.println("[BackgroundWorker] Executing asynchronous graceful shutdown callback...");
        callback.run(); // Notifies container that cleanup is complete!
    }

    @Override
    public boolean isRunning() {
        return running;
    }
}
```

**File:** `src/main/java/com/example/springcore/blog21/SmartLifecycleDemoApp.java`

```java
package com.example.springcore.blog21;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class SmartLifecycleDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog21.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 21 - SmartLifecycle & Graceful Shutdown Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            context.registerShutdownHook(); // Ensures JVM shutdown hook executes context.close()
            System.out.println("Application running... Closing context now.");
        }
    }
}
```

Expected Output:

```text
=== Blog 21 - SmartLifecycle & Graceful Shutdown Demo ===
[BackgroundWorker] Automatic start signal received! Starting worker thread...
Application running... Closing context now.
[BackgroundWorker] Stop signal received!
[BackgroundWorker] Executing asynchronous graceful shutdown callback...
```

---

## 5. Key Takeaways

1. `SmartLifecycle` components start automatically when `ApplicationContext` finishes refresh.
2. `getPhase()` dictates startup order (lower phase values start first and stop last).
3. Always invoke `context.registerShutdownHook()` in standalone applications to guarantee graceful shutdown.

---

## 6. Next Blog

**Part 22: `BeanPostProcessor`**
We explore `BeanPostProcessor`, modifying and wrapping bean instances before and after initialization.

Where you are: **Blog 21 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
