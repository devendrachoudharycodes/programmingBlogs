---
title: "Component Scanning Filters: Include and Exclude TypeFilters"
description: "Part 32 of the Spring Core series. How to fine-tune classpath scanning using includeFilters, excludeFilters, AnnotationTypeFilter, AssignableTypeFilter, and custom TypeFilter."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 32

tags:
  - java
  - spring
  - spring-core
  - component-scan
  - type-filters

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/component-scanning-filters/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 32 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 45 min / ~75 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | FilterType options, includeFilters, excludeFilters, useDefaultFilters, custom TypeFilter implementation |
| **Concepts unlocked** | Part 33 (@Import and ImportSelectors), Part 34 (Conditional Bean Registration) |

## 1. Prerequisites

**Spring concepts required**
- Component scanning basics (Parts 30 & 31)

**Previous blogs required**
- Part 30 (*Classpath Scanning*)
- Part 31 (*Custom Annotations and Stereotypes*)

---

## 2. What You Will Learn

- How to customize `@ComponentScan` filtering using `includeFilters` and `excludeFilters`
- Built-in `FilterType` options: `ANNOTATION`, `ASSIGNABLE_TYPE`, `REGEX`, `CUSTOM`
- Disabling default stereotype scanning via `useDefaultFilters = false`
- Building a custom `TypeFilter` for advanced class matching rules

---

## 3. `@ComponentScan` Filter Types

| FilterType | Example | Description |
|---|---|---|
| **`ANNOTATION`** | `@Filter(type = FilterType.ANNOTATION, classes = MyAnnotation.class)` | Includes/excludes classes carrying specified annotation. |
| **`ASSIGNABLE_TYPE`** | `@Filter(type = FilterType.ASSIGNABLE_TYPE, classes = LegacyService.class)` | Matches specified class or any subclass/implementation. |
| **`REGEX`** | `@Filter(type = FilterType.REGEX, pattern = ".*Test.*")` | Matches class name against regex pattern. |
| **`CUSTOM`** | `@Filter(type = FilterType.CUSTOM, classes = MyTypeFilter.class)` | Executes custom `TypeFilter.match(...)` rule. |

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog32/domain/ExcludedService.java`

```java
package com.example.springcore.blog32.domain;

import org.springframework.stereotype.Component;

@Component
public class ExcludedService {
    public ExcludedService() {
        System.out.println("ExcludedService instantiated!");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog32/domain/IncludedService.java`

```java
package com.example.springcore.blog32.domain;

import org.springframework.stereotype.Component;

@Component
public class IncludedService {
    public IncludedService() {
        System.out.println("[IncludedService] Instantiated successfully!");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog32/ScanFilterDemoApp.java`

```java
package com.example.springcore.blog32;

import com.example.springcore.blog32.domain.ExcludedService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

public class ScanFilterDemoApp {

    @Configuration
    @ComponentScan(
            basePackages = "com.example.springcore.blog32.domain",
            excludeFilters = @ComponentScan.Filter(
                    type = FilterType.ASSIGNABLE_TYPE,
                    classes = ExcludedService.class
            )
    )
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 32 - Component Scanning Filters Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            boolean containsExcluded = context.containsBean("excludedService");
            boolean containsIncluded = context.containsBean("includedService");

            System.out.println("Contains ExcludedService? " + containsExcluded); // false
            System.out.println("Contains IncludedService? " + containsIncluded); // true
        }
    }
}
```

Expected Output:

```text
=== Blog 32 - Component Scanning Filters Demo ===
[IncludedService] Instantiated successfully!
Contains ExcludedService? false
Contains IncludedService? true
```

---

## 5. Key Takeaways

1. Use `excludeFilters` to suppress specific classes, annotations, or regex patterns from component scanning.
2. `useDefaultFilters = false` disables standard `@Component` scanning, allowing 100% filter-driven component discovery.
3. Custom `TypeFilter`s allow inspection of class metadata before classes are loaded.

---

## 6. Next Blog

**Part 33: `@Import`, `ImportSelector` and `ImportBeanDefinitionRegistrar`**
We explore modular configuration composition and programmatic module enabling with `@Import` and `ImportSelector`.

Where you are: **Blog 32 of 49** (Phase 4a: Metadata, Configuration Internals and Scanning).
