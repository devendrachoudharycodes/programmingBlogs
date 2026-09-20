---
title: "Conditional Bean Registration: Custom Conditions and @Conditional"
description: "Part 34 of the Spring Core series. How @Conditional and the Condition interface evaluate runtime environment state to register or skip bean definitions."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 34

tags:
  - java
  - spring
  - spring-core
  - conditional-beans
  - conditions

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/conditional-bean-registration/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 34 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 35 min / 50 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | @Conditional annotation, Condition interface, ConditionContext, AnnotatedTypeMetadata, custom conditions |
| **Concepts unlocked** | Part 35 (Environment Abstraction), Part 36 (@Value and Property Resolution) |

## 1. Prerequisites

**Spring concepts required**
- `@Configuration`, `@Bean`, and `@Import` mechanisms (Parts 29 & 33)

**Previous blogs required**
- Part 29 (*@Configuration and @Bean Semantics*)
- Part 33 (*@Import, ImportSelector, and ImportBeanDefinitionRegistrar*)

---

## 2. What You Will Learn

- How `@Conditional` controls whether a bean definition is registered in the container
- Implementing custom conditions with the `Condition` interface
- Inspecting environment properties and system state via `ConditionContext`
- How `@Profile` is implemented internally as a meta-annotation over `@Conditional`

---

## 3. The `Condition` Interface

```java
public interface Condition {
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

If `matches(...)` returns `true`, the bean or configuration class is registered. If `false`, Spring completely ignores the bean definition!

---

## 4. Complete Working Example (Custom Operating System Condition)

**File:** `src/main/java/com/example/springcore/blog34/condition/OnLinuxCondition.java`

```java
package com.example.springcore.blog34.condition;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class OnLinuxCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String osName = context.getEnvironment().getProperty("os.name", "");
        return osName.toLowerCase().contains("linux");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog34/domain/LinuxScriptEngine.java`

```java
package com.example.springcore.blog34.domain;

import com.example.springcore.blog34.condition.OnLinuxCondition;
import org.springframework.context.annotation.Conditional;
import org.springframework.stereotype.Component;

@Component
@Conditional(OnLinuxCondition.class) // Only loaded if OS is Linux!
public class LinuxScriptEngine {
    public void run() {
        System.out.println("[LinuxScriptEngine] Running bash script...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog34/ConditionalDemoApp.java`

```java
package com.example.springcore.blog34;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class ConditionalDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog34.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 34 - Conditional Bean Registration Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            boolean loaded = context.containsBean("linuxScriptEngine");
            System.out.println("OS Name: " + System.getProperty("os.name"));
            System.out.println("Was LinuxScriptEngine loaded? " + loaded);
        }
    }
}
```

Expected Output (on Linux):

```text
=== Blog 34 - Conditional Bean Registration Demo ===
OS Name: Linux
Was LinuxScriptEngine loaded? true
```

---

## 5. Key Takeaways

1. `@Conditional` evaluates runtime conditions before registering bean definitions.
2. `ConditionContext` provides access to `Environment`, `ResourceLoader`, and `BeanDefinitionRegistry`.
3. XML Configuration does not support `@Conditional` directly; profile-based `<beans profile="...">` or property placeholders are used instead.

---

## 6. Next Blog

**Part 35: Environment Abstraction**
We explore profiles, properties, property sources, `Environment`, and profile-specific bean configuration.

Where you are: **Blog 34 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning - Complete!).
