---
title: "Programmatic Registration and BeanDefinitionRegistry"
description: "Part 27 of the Spring Core series. How to programmatically register beans using BeanDefinitionRegistry, BeanDefinitionBuilder, and Supplier callbacks."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 27

tags:
  - java
  - spring
  - spring-core
  - bean-definition-registry
  - programmatic-registration

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/programmatic-registration-and-beandefinitionregistry/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 27 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 35 min / 50 min / ~85 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | BeanDefinitionRegistry, BeanDefinitionBuilder, Supplier-based bean registration, BeanDefinitionRegistryPostProcessor |
| **Concepts unlocked** | Part 28 (Annotation Processing Infrastructure), Part 33 (@Import and ImportSelectors) |

## 1. Prerequisites

**Spring concepts required**
- `BeanDefinition` metadata and container extension points (Parts 23 & 26)

**Previous blogs required**
- Part 23 (*Container Extension Points*)
- Part 26 (*BeanDefinition*)

---

## 2. What You Will Learn

- How to programmatically register beans without `@Bean`, `@Component`, or XML
- Using `BeanDefinitionRegistry` and `BeanDefinitionBuilder`
- Functional Supplier-based bean registration via `GenericApplicationContext.registerBean(...)`
- Implementing `BeanDefinitionRegistryPostProcessor` to dynamically register beans during startup

---

## 3. Programmatic Registration API

Spring 5+ / 6+ / 7.0 provides functional bean registration via `GenericApplicationContext`:

```java
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean("myService", MyService.class, () -> new MyService(dep));
context.refresh();
```

Or via `BeanDefinitionRegistryPostProcessor`:

```java
BeanDefinition bd = BeanDefinitionBuilder.genericBeanDefinition(MyService.class)
        .setScope("singleton")
        .setLazyInit(false)
        .getBeanDefinition();

registry.registerBeanDefinition("myService", bd);
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog27/domain/DynamicService.java`

```java
package com.example.springcore.blog27.domain;

public class DynamicService {
    private final String configName;

    public DynamicService(String configName) {
        this.configName = configName;
    }

    public void execute() {
        System.out.println("[DynamicService] Executing with config: " + configName);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog27/ProgrammaticDemoApp.java`

```java
package com.example.springcore.blog27;

import com.example.springcore.blog27.domain.DynamicService;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ProgrammaticDemoApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 27 - Programmatic Registration Demo ===");

        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {

            // 1. Register Bean via Supplier (Functional Registration)
            context.registerBean("dynamicSupplierService", DynamicService.class,
                    () -> new DynamicService("Supplier-Config"));

            // 2. Register Bean via BeanDefinitionBuilder
            context.registerBeanDefinition("dynamicBuilderService",
                    BeanDefinitionBuilder.genericBeanDefinition(DynamicService.class)
                            .addConstructorArgValue("Builder-Config")
                            .getBeanDefinition()
            );

            // Refresh context AFTER registration
            context.refresh();

            // Retrieve and test beans
            DynamicService s1 = context.getBean("dynamicSupplierService", DynamicService.class);
            s1.execute();

            DynamicService s2 = context.getBean("dynamicBuilderService", DynamicService.class);
            s2.execute();
        }
    }
}
```

Expected Output:

```text
=== Blog 27 - Programmatic Registration Demo ===
[DynamicService] Executing with config: Supplier-Config
[DynamicService] Executing with config: Builder-Config
```

---

## 5. Key Takeaways

1. `GenericApplicationContext.registerBean(...)` offers clean functional bean registration.
2. `BeanDefinitionBuilder` builds `BeanDefinition` instances programmatically.
3. Programmatic registration allows dynamic bean generation at runtime without reflection component scanning overhead.

---

## 6. Next Blog

**Part 28: Annotation Processing Infrastructure**
We explore how Spring's internal post-processors (`ConfigurationClassPostProcessor`, `AutowiredAnnotationBeanPostProcessor`) process annotations.

Where you are: **Blog 27 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
