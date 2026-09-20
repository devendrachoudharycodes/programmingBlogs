---
title: "Annotation Processing Infrastructure: The Machinery Behind Annotations"
description: "Part 28 of the Spring Core series. How internal post-processors like ConfigurationClassPostProcessor and AutowiredAnnotationBeanPostProcessor convert annotations into behavior."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 28

tags:
  - java
  - spring
  - spring-core
  - annotation-processors
  - internals

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/annotation-processing-infrastructure/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 28 of 49 |
| **Difficulty** | ⚫ Internals / Architecture |
| **Reading / Coding / Total** | 35 min / 35 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Internal annotation infrastructure beans, ConfigurationClassPostProcessor, AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor |
| **Concepts unlocked** | Part 29 (@Configuration and @Bean Semantics), Part 30 (Classpath Scanning) |

## 1. Prerequisites

**Spring concepts required**
- `BeanPostProcessor`, `BeanFactoryPostProcessor`, and `BeanDefinition` (Parts 22, 23 & 26)

**Previous blogs required**
- Part 22 (*BeanPostProcessor*)
- Part 23 (*Container Extension Points*)

---

## 2. What You Will Learn

- Why annotations like `@Autowired`, `@PostConstruct`, and `@Configuration` do nothing on their own
- How Spring registers internal infrastructure post-processors to interpret annotations
- The roles of key infrastructure beans: `ConfigurationClassPostProcessor`, `AutowiredAnnotationBeanPostProcessor`, and `CommonAnnotationBeanPostProcessor` *(implementation details)*
- How to inspect internal infrastructure beans in a running `ApplicationContext`

---

## 3. The Annotation Machinery

Annotations in Java are passive metadata. Spring processes them by registering dedicated infrastructure post-processors:

```text
Annotation                Infrastructure Post-Processor (Implementation Detail)
────────────────────────  ──────────────────────────────────────────────────────────────────
@Configuration, @Bean ──► ConfigurationClassPostProcessor (BeanFactoryPostProcessor)
@Autowired, @Value    ──► AutowiredAnnotationBeanPostProcessor (BeanPostProcessor)
@PostConstruct        ──► CommonAnnotationBeanPostProcessor (BeanPostProcessor)
```

---

## 4. Complete Working Example (Observing Infrastructure Beans)

**File:** `src/main/java/com/example/springcore/blog28/AnnotationInfrastructureDemoApp.java`

```java
package com.example.springcore.blog28;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;

public class AnnotationInfrastructureDemoApp {

    @Configuration
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 28 - Observing Internal Infrastructure Beans ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("--- All Registered Bean Names (Including Infrastructure) ---");
            String[] names = context.getBeanDefinitionNames();
            Arrays.stream(names).forEach(name ->
                    System.out.println("Bean Name: " + name + " | Class: " + context.getBean(name).getClass().getSimpleName())
            );
        }
    }
}
```

Expected Output:

```text
=== Blog 28 - Observing Internal Infrastructure Beans ===
--- All Registered Bean Names (Including Infrastructure) ---
Bean Name: org.springframework.context.annotation.internalConfigurationAnnotationProcessor | Class: ConfigurationClassPostProcessor
Bean Name: org.springframework.context.annotation.internalAutowiredAnnotationProcessor | Class: AutowiredAnnotationBeanPostProcessor
Bean Name: org.springframework.context.annotation.internalCommonAnnotationProcessor | Class: CommonAnnotationBeanPostProcessor
Bean Name: annotationInfrastructureDemoApp.Config | Class: Config$$SpringCGLIB$$0
```

---

## 5. Key Takeaways

1. Annotations are passive metadata; post-processor infrastructure beans perform the actual processing.
2. `ConfigurationClassPostProcessor` processes `@Configuration`, `@Bean`, and `@Import`.
3. `AutowiredAnnotationBeanPostProcessor` injects `@Autowired` fields and constructors.

---

## 6. Next Blog

**Part 29: `@Configuration` and `@Bean` Semantics**
We explore full vs lite mode, CGLIB enhancement of configuration classes, and `proxyBeanMethods`.

Where you are: **Blog 28 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
