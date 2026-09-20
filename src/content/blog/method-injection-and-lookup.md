---
title: "Method Injection and @Lookup: Injecting Prototypes into Singletons"
description: "Part 16 of the Spring Core series. Solving the singleton-prototype scope mismatch problem using Spring @Lookup method injection and CGLIB subclassing."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 16

tags:
  - java
  - spring
  - spring-core
  - method-injection
  - lookup-annotation

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/method-injection-and-lookup/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 16 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Scope mismatch problem, method injection, @Lookup annotation, CGLIB subclass bytecode generation |
| **Concepts unlocked** | Part 17 (ObjectProvider), Part 18 (Custom Bean Scopes) |

## 1. Prerequisites

**Spring concepts required**
- Singleton vs Prototype scopes (Part 15)

**Previous blogs required**
- Part 15 (*Bean Scopes*)

---

## 2. What You Will Learn

- The **Singleton-Prototype Scope Mismatch Problem**: Why direct injection of a prototype into a singleton fails
- How Spring's `@Lookup` method injection solves the scope mismatch
- How CGLIB dynamically subclasses `@Lookup` annotated classes at runtime *(implementation detail)*
- XML `<lookup-method>` equivalent syntax

---

## 3. The Scope Mismatch Problem

When a `singleton` bean (`OrderProcessor`) directly injects a `prototype` bean (`CommandTask`), Spring injects `CommandTask` **once** when `OrderProcessor` is created. Subsequent invocations of `OrderProcessor` keep reusing that **same initial prototype instance**!

```text
Singleton OrderProcessor (Created Once at Startup)
       │
       └── Injected Prototype CommandTask (Injected ONCE during startup!)
```

---

## 4. The `@Lookup` Solution

By annotating a method with `@Lookup`, Spring overrides the method using CGLIB bytecode generation to fetch a fresh `prototype` bean from the `ApplicationContext` on every call:

```java
@Component
public abstract class OrderProcessor {

    public void process() {
        CommandTask task = createCommandTask(); // Always returns a NEW prototype instance!
        task.execute();
    }

    @Lookup
    protected abstract CommandTask createCommandTask(); // Overridden dynamically by Spring CGLIB
}
```

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog16/domain/CommandTask.java`

```java
package com.example.springcore.blog16.domain;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototype")
public class CommandTask {
    public void execute(int taskId) {
        System.out.println("[CommandTask #" + taskId + "] Executing task instance: " + System.identityHashCode(this));
    }
}
```

**File:** `src/main/java/com/example/springcore/blog16/domain/OrderProcessor.java`

```java
package com.example.springcore.blog16.domain;

import org.springframework.beans.factory.annotation.Lookup;
import org.springframework.stereotype.Component;

@Component
public abstract class OrderProcessor {

    public void processOrder(int id) {
        CommandTask task = getCommandTask();
        task.execute(id);
    }

    // Spring overrides this stub method using CGLIB proxying
    @Lookup
    public abstract CommandTask getCommandTask();
}
```

**File:** `src/main/java/com/example/springcore/blog16/LookupDemoApp.java`

```java
package com.example.springcore.blog16;

import com.example.springcore.blog16.domain.OrderProcessor;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class LookupDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog16.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 16 - @Lookup Method Injection Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            OrderProcessor processor = context.getBean(OrderProcessor.class);
            System.out.println("Processor Class: " + processor.getClass().getName());

            processor.processOrder(101);
            processor.processOrder(102);
            processor.processOrder(103);
        }
    }
}
```

Expected Output:

```text
=== Blog 16 - @Lookup Method Injection Demo ===
Processor Class: com.example.springcore.blog16.domain.OrderProcessor$$SpringCGLIB$$0
[CommandTask #101] Executing task instance: 187212456
[CommandTask #102] Executing task instance: 154823901
[CommandTask #103] Executing task instance: 991823122
```

Notice each invocation received a **brand-new prototype instance** (different identity hash codes)!

---

## 6. Key Takeaways

1. Direct injection of a prototype into a singleton freezes the prototype instance.
2. `@Lookup` overrides a stub method to perform a dynamic lookup every time it is called.
3. Spring uses CGLIB bytecode enhancement to subclass classes containing `@Lookup` methods.

---

## 7. Next Blog

**Part 17: `ObjectProvider`, `ObjectFactory` and `Provider`**
We explore modern programmatic alternatives to `@Lookup` for lazy and prototype dependency resolution.

Where you are: **Blog 16 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
