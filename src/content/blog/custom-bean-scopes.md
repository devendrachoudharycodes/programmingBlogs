---
title: "Custom Bean Scopes: Implementing the Scope Interface"
description: "Part 18 of the Spring Core series. How to build and register a custom Spring bean scope from scratch by implementing org.springframework.beans.factory.config.Scope."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 18

tags:
  - java
  - spring
  - spring-core
  - custom-scopes
  - bean-scopes

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/custom-bean-scopes/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 18 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 35 min / 50 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | org.springframework.beans.factory.config.Scope interface, CustomScopeConfigurer, ThreadLocal-based scope storage |
| **Concepts unlocked** | Part 19 (Complete Bean Lifecycle), Part 23 (Container Extension Points) |

## 1. Prerequisites

**Spring concepts required**
- Singleton vs Prototype bean scopes (Part 15)

**Previous blogs required**
- Part 15 (*Bean Scopes*)
- Part 17 (*ObjectProvider, ObjectFactory, and Provider*)

---

## 2. What You Will Learn

- The structure and methods of the `org.springframework.beans.factory.config.Scope` interface
- How to manage contextual object lifecycle, creation, and destruction callbacks
- How to register custom scopes using `CustomScopeConfigurer`
- How to build a custom `ThreadScope` that manages one bean instance per executing thread

---

## 3. The `Scope` Interface API

To build a custom scope, implement `org.springframework.beans.factory.config.Scope`:

```java
public interface Scope {
    Object get(String name, ObjectFactory<?> objectFactory);
    Object remove(String name);
    void registerDestructionCallback(String name, Runnable callback);
    Object resolveContextualObject(String key);
    String getConversationId();
}
```

---

## 4. Complete Working Example (Custom ThreadScope)

**File:** `src/main/java/com/example/springcore/blog18/scope/SimpleThreadScope.java`

```java
package com.example.springcore.blog18.scope;

import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.Scope;

import java.util.HashMap;
import java.util.Map;

public class SimpleThreadScope implements Scope {

    private final ThreadLocal<Map<String, Object>> threadScope =
            ThreadLocal.withInitial(HashMap::new);

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadScope.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        return threadScope.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Simple thread scope cleanup callback
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```

**File:** `src/main/java/com/example/springcore/blog18/domain/TenantContext.java`

```java
package com.example.springcore.blog18.domain;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("thread") // Custom Scope Name!
public class TenantContext {
    private final String threadName = Thread.currentThread().getName();

    public void printInfo() {
        System.out.println("[TenantContext] Thread: " + threadName + " | ID: " + System.identityHashCode(this));
    }
}
```

**File:** `src/main/java/com/example/springcore/blog18/CustomScopeDemoApp.java`

```java
package com.example.springcore.blog18;

import com.example.springcore.blog18.domain.TenantContext;
import com.example.springcore.blog18.scope.SimpleThreadScope;
import org.springframework.beans.factory.config.CustomScopeConfigurer;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.util.Collections;

public class CustomScopeDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog18.domain")
    static class Config {

        @Bean
        public static CustomScopeConfigurer customScopeConfigurer() {
            CustomScopeConfigurer configurer = new CustomScopeConfigurer();
            configurer.setScopes(Collections.singletonMap("thread", new SimpleThreadScope()));
            return configurer;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("=== Blog 18 - Custom Bean Scope Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            Runnable task = () -> {
                TenantContext t1 = context.getBean(TenantContext.class);
                TenantContext t2 = context.getBean(TenantContext.class);
                t1.printInfo();
                System.out.println("Same Instance in same thread? " + (t1 == t2));
            };

            Thread thread1 = new Thread(task, "Worker-Thread-1");
            Thread thread2 = new Thread(task, "Worker-Thread-2");

            thread1.start();
            thread1.join();

            thread2.start();
            thread2.join();
        }
    }
}
```

Expected Output:

```text
=== Blog 18 - Custom Bean Scope Demo ===
[TenantContext] Thread: Worker-Thread-1 | ID: 198231024
Same Instance in same thread? true
[TenantContext] Thread: Worker-Thread-2 | ID: 882910391
Same Instance in same thread? true
```

---

## 5. Key Takeaways

1. Implement `org.springframework.beans.factory.config.Scope` to create custom lifecycle boundaries.
2. Register custom scopes using a `static` `CustomScopeConfigurer` bean in Java Configuration.
3. ThreadLocal is the standard storage strategy for thread-bound custom scopes.

---

## 6. Next Blog

**Part 19: Complete Bean Lifecycle**
We build a master lifecycle bean demonstrating all 12+ initialization and destruction hooks in sequence.

Where you are: **Blog 18 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
