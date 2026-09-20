---
title: "Singleton Registry and the Three-Level Cache: DefaultSingletonBeanRegistry Internals"
description: "Part 46 of the Spring Core series. Inside DefaultSingletonBeanRegistry, singletonObjects, earlySingletonObjects, and singletonFactories."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 46

tags:
  - java
  - spring
  - spring-core
  - singleton-registry
  - three-level-cache

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/singleton-registry-and-the-three-level-cache/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 46 of 49 |
| **Difficulty** | ⚫ Internals / Architecture |
| **Reading / Coding / Total** | 45 min / 45 min / ~90 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | DefaultSingletonBeanRegistry, singletonObjects, earlySingletonObjects, singletonFactories, early singleton exposure |
| **Concepts unlocked** | Part 47 (Dependency Resolution Internals), Part 48 (Container Error Analysis) |

## 1. Prerequisites

**Spring concepts required**
- Circular dependencies and container refresh process (Parts 14 & 45)

**Previous blogs required**
- Part 14 (*Circular Dependencies*)
- Part 45 (*Spring Container Initialization*)

---

## 2. What You Will Learn

- How `DefaultSingletonBeanRegistry` stores, manages, and retrieves singleton instances
- The exact role of each level in the **Three-Level Singleton Cache** *(implementation details)*:
  - `singletonObjects` (Level 1)
  - `earlySingletonObjects` (Level 2)
  - `singletonFactories` (Level 3)
- How early references enable circular dependency resolution without creating duplicate proxy instances

---

## 3. The Three-Level Cache Data Structures

Inside `DefaultSingletonBeanRegistry`:

```java
// Level 1: Fully initialized singletons ready for application use
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// Level 2: Early raw/proxied singleton references (partially initialized)
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);

// Level 3: Factories providing early singleton references
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

---

## 4. Cache Promotion Flow (`getSingleton`)

```text
Request Bean Name "a"
       │
       ▼
Check Level 1 (singletonObjects) ──FOUND──► Return Ready Singleton
       │
   NOT FOUND
       ▼
Check Level 2 (earlySingletonObjects) ──FOUND──► Return Early Reference
       │
   NOT FOUND & Is Bean Currently in Creation?
       │
      YES
       ▼
Check Level 3 (singletonFactories) ──FOUND──► Call Factory.getObject()
                                            │ Promote Reference to Level 2
                                            │ Remove Factory from Level 3
                                            └─► Return Early Reference
```

---

## 5. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog46/SingletonRegistryDemoApp.java`

```java
package com.example.springcore.blog46;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Configuration;

public class SingletonRegistryDemoApp {

    @Configuration
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 46 - Singleton Registry Internals Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("Singleton Count in Context: " + context.getBeanDefinitionCount());
            System.out.println("Contains 'singletonRegistryDemoApp.Config'? "
                    + context.containsBean("singletonRegistryDemoApp.Config"));
        }
    }
}
```

Expected Output:

```text
=== Blog 46 - Singleton Registry Internals Demo ===
Singleton Count in Context: 6
Contains 'singletonRegistryDemoApp.Config'? true
```

---

## 6. Key Takeaways

1. `DefaultSingletonBeanRegistry` is the concrete base class managing singleton lifecycle storage.
2. Level 1 stores finished singletons, Level 2 stores early references, Level 3 stores early factories.
3. Level 3 factories allow AOP post-processors to generate early proxies when circular references occur.

---

## 7. Next Blog

**Part 47: Dependency Resolution Internals**
We explore how `DefaultListableBeanFactory.doResolveDependency()` matches target injection parameters to candidates.

Where you are: **Blog 46 of 49** (Phase 5: Architecture, Internals, and Diagnostics).
