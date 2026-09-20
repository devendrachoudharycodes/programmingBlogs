---
title: "Dependency Resolution Internals: Inside doResolveDependency()"
description: "Part 47 of the Spring Core series. Inside DefaultListableBeanFactory.doResolveDependency(), DependencyDescriptor, and the candidate selection algorithm."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 47

tags:
  - java
  - spring
  - spring-core
  - dependency-resolution
  - internals

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/dependency-resolution-internals/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 47 of 49 |
| **Difficulty** | ⚫ Internals / Architecture |
| **Reading / Coding / Total** | 45 min / 50 min / ~95 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | DefaultListableBeanFactory.doResolveDependency(), DependencyDescriptor, candidate matching, Primary/Qualifier filtering |
| **Concepts unlocked** | Part 48 (Container Error Analysis), Part 49 (Spring Core Design Principles) |

## 1. Prerequisites

**Spring concepts required**
- Autowiring, `@Primary`, `@Qualifier`, and `BeanDefinition` (Parts 10, 12 & 26)

**Previous blogs required**
- Part 12 (*@Primary and @Qualifier*)
- Part 26 (*BeanDefinition*)
- Part 46 (*Singleton Registry and the Three-Level Cache*)

---

## 2. What You Will Learn

- How `DefaultListableBeanFactory.doResolveDependency()` evaluates target injection points *(implementation detail)*
- The role of `DependencyDescriptor` in wrapping constructor/setter parameter metadata
- How Spring filters candidate beans by Type ──► Qualifiers ──► Primary ──► Bean Name
- How shortcut resolutions optimize dependency injection for singletons

---

## 3. Dependency Resolution Decision Tree

```text
Dependency Descriptor (Field / Parameter)
       │
       ▼
Find All Candidate Beans Matching Requested Type
       │
       ├── 0 Candidates ──► Is @Autowired(required = false)? ──► YES: Return null
       │                                                     └── NO : Throw NoSuchBeanDefinitionException
       │
       ├── 1 Candidate  ──► Return Candidate Bean
       │
       └── Multiple Candidates (Ambiguity)
               │
               ▼
       Filter Candidates by @Qualifier
               │
               ▼
       Check for Single @Primary Bean
               │
               ▼
       Check for Bean Name Matching Parameter Name
               │
               ├── Exactly 1 Match ──► Return Candidate
               └── Ambiguity Unresolved ──► Throw NoUniqueBeanDefinitionException!
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog47/domain/ServiceA.java`

```java
package com.example.springcore.blog47.domain;

import org.springframework.stereotype.Component;

@Component
public class ServiceA {
    public void execute() {
        System.out.println("[ServiceA] Executing...");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog47/domain/ClientService.java`

```java
package com.example.springcore.blog47.domain;

import org.springframework.stereotype.Component;

@Component
public class ClientService {
    private final ServiceA serviceA;

    public ClientService(ServiceA serviceA) {
        this.serviceA = serviceA;
    }

    public void run() {
        serviceA.execute();
    }
}
```

**File:** `src/main/java/com/example/springcore/blog47/ResolutionDemoApp.java`

```java
package com.example.springcore.blog47;

import com.example.springcore.blog47.domain.ClientService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class ResolutionDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog47.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 47 - Dependency Resolution Internals Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            ClientService client = context.getBean(ClientService.class);
            client.run();
        }
    }
}
```

Expected Output:

```text
=== Blog 47 - Dependency Resolution Internals Demo ===
[ServiceA] Executing...
```

---

## 5. Key Takeaways

1. `DependencyDescriptor` encapsulates injection target metadata (type, annotations, required status).
2. `doResolveDependency()` evaluates candidate beans against Type, Qualifiers, Primary, and Name.
3. Ambiguity failures occur when multiple candidates remain after all filtering stages.

---

## 6. Next Blog

**Part 48: Spring Container Error Analysis**
We analyze common Spring startup exceptions (`NoSuchBeanDefinitionException`, `NoUniqueBeanDefinitionException`, `BeanCurrentlyInCreationException`, `BeanDefinitionOverrideException`) and how to diagnose them.

Where you are: **Blog 47 of 49** (Phase 5: Architecture, Internals, and Diagnostics).
