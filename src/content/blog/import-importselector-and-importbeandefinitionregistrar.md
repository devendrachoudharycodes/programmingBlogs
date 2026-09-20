---
title: "@Import, ImportSelector, and ImportBeanDefinitionRegistrar"
description: "Part 33 of the Spring Core series. Building modular frameworks using @Import, dynamic ImportSelector, and ImportBeanDefinitionRegistrar."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 33

tags:
  - java
  - spring
  - spring-core
  - import-annotation
  - enable-pattern

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/import-importselector-and-importbeandefinitionregistrar/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 33 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 40 min / 55 min / ~95 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | @Import, ImportSelector, DeferredImportSelector, ImportBeanDefinitionRegistrar, building custom @EnableXxx annotations |
| **Concepts unlocked** | Part 34 (Conditional Bean Registration), Part 35 (Environment Abstraction) |

## 1. Prerequisites

**Spring concepts required**
- Programmatic bean registration and `@Configuration` semantics (Parts 27 & 29)

**Previous blogs required**
- Part 27 (*Programmatic Registration*)
- Part 29 (*@Configuration and @Bean Semantics*)

---

## 2. What You Will Learn

- How `@Import` modularizes configuration classes without component scanning
- How `ImportSelector` dynamically selects which configuration classes to load based on annotation metadata
- How `ImportBeanDefinitionRegistrar` programmatically adds bean definitions
- Building a custom `@EnableAuditLogging` framework annotation from scratch

---

## 3. The 3 Forms of `@Import`

```text
@Import(target)
   ├── Case 1: Plain @Configuration / Component class ──► Loaded directly
   ├── Case 2: Implements ImportSelector              ──► Returns String[] of class names to load
   └── Case 3: Implements ImportBeanDefinitionRegistrar──► Programmatically registers BeanDefinitions
```

---

## 4. Complete Working Example (Custom `@EnableAuditLogging`)

**File:** `src/main/java/com/example/springcore/blog33/domain/AuditService.java`

```java
package com.example.springcore.blog33.domain;

public class AuditService {
    public void log(String message) {
        System.out.println("[AuditService (@EnableAuditLogging)] " + message);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog33/config/AuditImportSelector.java`

```java
package com.example.springcore.blog33.config;

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

public class AuditImportSelector implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        System.out.println("[ImportSelector] Dynamically selecting AuditConfig class...");
        return new String[] { "com.example.springcore.blog33.config.AuditConfig" };
    }
}
```

**File:** `src/main/java/com/example/springcore/blog33/config/AuditConfig.java`

```java
package com.example.springcore.blog33.config;

import com.example.springcore.blog33.domain.AuditService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AuditConfig {

    @Bean
    public AuditService auditService() {
        return new AuditService();
    }
}
```

**File:** `src/main/java/com/example/springcore/blog33/annotation/EnableAuditLogging.java`

```java
package com.example.springcore.blog33.annotation;

import com.example.springcore.blog33.config.AuditImportSelector;
import org.springframework.context.annotation.Import;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AuditImportSelector.class) // Custom @Enable pattern!
public @interface EnableAuditLogging {
}
```

**File:** `src/main/java/com/example/springcore/blog33/ImportDemoApp.java`

```java
package com.example.springcore.blog33;

import com.example.springcore.blog33.annotation.EnableAuditLogging;
import com.example.springcore.blog33.domain.AuditService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

public class ImportDemoApp {

    @Configuration
    @EnableAuditLogging // Enables audit framework module!
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 33 - @Import & @Enable Pattern Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            AuditService service = context.getBean(AuditService.class);
            service.log("System checkout initiated.");
        }
    }
}
```

Expected Output:

```text
=== Blog 33 - @Import & @Enable Pattern Demo ===
[ImportSelector] Dynamically selecting AuditConfig class...
[AuditService (@EnableAuditLogging)] System checkout initiated.
```

---

## 5. Key Takeaways

1. `@Import` explicitly imports configuration classes without package scanning.
2. `ImportSelector` selects class names dynamically based on annotation attributes.
3. Custom `@EnableXxx` annotations combine `@Import` with `ImportSelector` or `ImportBeanDefinitionRegistrar`.

---

## 6. Next Blog

**Part 34: Conditional Bean Registration**
We explore `@Conditional`, `Condition`, `ConditionContext`, `@Profile`, and custom conditions.

Where you are: **Blog 33 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
