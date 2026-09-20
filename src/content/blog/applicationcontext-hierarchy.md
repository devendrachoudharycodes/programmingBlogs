---
title: "ApplicationContext Hierarchy: Parent-Child Contexts and Bean Visibility"
description: "Part 44 of the Spring Core series. How parent-child ApplicationContext hierarchies enforce bean visibility rules, isolation, and inheritance."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 44

tags:
  - java
  - spring
  - spring-core
  - context-hierarchy
  - parent-child

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/applicationcontext-hierarchy/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 44 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Parent-child ApplicationContext, setParent(), bean visibility rules, context isolation |
| **Concepts unlocked** | Part 45 (Refresh Process), Part 48 (Container Error Analysis) |

## 1. Prerequisites

**Spring concepts required**
- `ApplicationContext` implementations and bean lookups (Parts 6 & 43)

**Previous blogs required**
- Part 42 (*ApplicationContext Internal Architecture*)
- Part 43 (*ApplicationContext Implementations*)

---

## 2. What You Will Learn

- How parent-child `ApplicationContext` hierarchies structure complex applications
- **Bean Visibility Rules**: Child contexts can see and inject beans from Parent contexts, but Parent contexts **CANNOT** see Child beans!
- Overriding parent beans inside child contexts
- Historical & architecture use cases (Spring MVC `DispatcherServlet` child context over root ApplicationContext)

---

## 3. Hierarchy Visibility Rules

```text
 ┌────────────────────────────────────────────────────────┐
 │   Parent Context (Root: Services, Repositories)        │
 │   - SharedServiceBean                                  │
 └───────────────────────────▲────────────────────────────┘
                             │ Child can see Parent
                             │ Parent CANNOT see Child
 ┌───────────────────────────┴────────────────────────────┐
 │   Child Context (Web: Controllers, Views)              │
 │   - WebControllerBean                                  │
 └────────────────────────────────────────────────────────┘
```

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog44/domain/SharedParentBean.java`

```java
package com.example.springcore.blog44.domain;

public class SharedParentBean {
    public void execute() {
        System.out.println("[SharedParentBean] Executing from Parent Context");
    }
}
```

**File:** `src/main/java/com/example/springcore/blog44/domain/ChildWebBean.java`

```java
package com.example.springcore.blog44.domain;

public class ChildWebBean {
    private final SharedParentBean parentBean;

    public ChildWebBean(SharedParentBean parentBean) {
        this.parentBean = parentBean;
    }

    public void run() {
        System.out.print("[ChildWebBean] Calling Parent Bean: ");
        parentBean.execute();
    }
}
```

**File:** `src/main/java/com/example/springcore/blog44/HierarchyDemoApp.java`

```java
package com.example.springcore.blog44;

import com.example.springcore.blog44.domain.ChildWebBean;
import com.example.springcore.blog44.domain.SharedParentBean;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HierarchyDemoApp {

    public static void main(String[] args) {
        System.out.println("=== Blog 44 - ApplicationContext Hierarchy Demo ===");

        // 1. Initialize Parent Context
        AnnotationConfigApplicationContext parentContext = new AnnotationConfigApplicationContext();
        parentContext.registerBean("sharedParentBean", SharedParentBean.class, SharedParentBean::new);
        parentContext.refresh();

        // 2. Initialize Child Context linked to Parent
        AnnotationConfigApplicationContext childContext = new AnnotationConfigApplicationContext();
        childContext.setParent(parentContext); // Establish Parent-Child Hierarchy!
        childContext.registerBean("childWebBean", ChildWebBean.class,
                () -> new ChildWebBean(childContext.getBean(SharedParentBean.class)));
        childContext.refresh();

        // 3. Child can lookup parent bean
        System.out.println("Child can see parent bean? " + childContext.containsBean("sharedParentBean")); // true

        // 4. Parent CANNOT see child bean
        System.out.println("Parent can see child bean? " + parentContext.containsBean("childWebBean")); // false

        // 5. Execute child bean
        ChildWebBean childBean = childContext.getBean(ChildWebBean.class);
        childBean.run();

        childContext.close();
        parentContext.close();
    }
}
```

Expected Output:

```text
=== Blog 44 - ApplicationContext Hierarchy Demo ===
Child can see parent bean? true
Parent can see child bean? false
[ChildWebBean] Calling Parent Bean: [SharedParentBean] Executing from Parent Context
```

---

## 5. Key Takeaways

1. Child contexts can query and inject beans defined in parent contexts.
2. Parent contexts have zero visibility into child context beans.
3. Useful for isolating web controllers from core domain services.

---

## 6. Next Blog

**Part 45: Spring Container Initialization (the `refresh` process)**
We perform a complete step-by-step deep dive into `AbstractApplicationContext.refresh()`.

Where you are: **Blog 44 of 49** (Phase 5: Architecture, Internals, and Diagnostics).
