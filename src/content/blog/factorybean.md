---
title: "FactoryBean: Complex Object Construction and the Dereference Operator"
description: "Part 25 of the Spring Core series. How org.springframework.beans.factory.FactoryBean constructs complex objects and how the & prefix accesses the factory itself."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 25

tags:
  - java
  - spring
  - spring-core
  - factory-bean
  - complex-objects

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/factorybean/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 25 of 49 |
| **Difficulty** | 🟡 Intermediate |
| **Reading / Coding / Total** | 30 min / 40 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | FactoryBean contract, getObject(), getObjectType(), isSingleton(), dereference operator (&) |
| **Concepts unlocked** | Part 26 (BeanDefinition), Part 27 (Programmatic Registration) |

## 1. Prerequisites

**Spring concepts required**
- Beans, `ApplicationContext`, and container extension points (Parts 3 & 23)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 23 (*Container Extension Points*)

---

## 2. What You Will Learn

- What a `FactoryBean<T>` is and why it exists for complex or legacy object construction
- The 3 methods of `FactoryBean<T>`: `getObject()`, `getObjectType()`, and `isSingleton()`
- The difference between `context.getBean("myBean")` (returns product) and `context.getBean("&myBean")` (returns factory)
- `FactoryBean` vs `@Bean` factory methods

---

## 3. `FactoryBean<T>` Contract

```java
public interface FactoryBean<T> {
    T getObject() throws Exception;
    Class<?> getObjectType();
    default boolean isSingleton() { return true; }
}
```

When a bean implements `FactoryBean<T>`, Spring registers the `FactoryBean` under its name, but `context.getBean("name")` returns the **produced object** (`T`), NOT the `FactoryBean` instance.

To retrieve the actual `FactoryBean` instance, prefix the bean name with `&`: `context.getBean("&name")`.

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog25/domain/DatabaseConnection.java`

```java
package com.example.springcore.blog25.domain;

public class DatabaseConnection {
    private final String url;

    public DatabaseConnection(String url) {
        this.url = url;
    }

    public void connect() {
        System.out.println("[DatabaseConnection] Connected to " + url);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog25/domain/DatabaseConnectionFactoryBean.java`

```java
package com.example.springcore.blog25.domain;

import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

@Component("dbConnection") // Bean Name = "dbConnection"
public class DatabaseConnectionFactoryBean implements FactoryBean<DatabaseConnection> {

    @Override
    public DatabaseConnection getObject() throws Exception {
        System.out.println("[DatabaseConnectionFactoryBean] Constructing complex DatabaseConnection...");
        return new DatabaseConnection("jdbc:postgresql://localhost:5432/mydb");
    }

    @Override
    public Class<?> getObjectType() {
        return DatabaseConnection.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

**File:** `src/main/java/com/example/springcore/blog25/FactoryBeanDemoApp.java`

```java
package com.example.springcore.blog25;

import com.example.springcore.blog25.domain.DatabaseConnection;
import com.example.springcore.blog25.domain.DatabaseConnectionFactoryBean;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class FactoryBeanDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog25.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 25 - FactoryBean Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            // 1. Fetch produced product (DatabaseConnection)
            DatabaseConnection conn = context.getBean("dbConnection", DatabaseConnection.class);
            conn.connect();

            // 2. Fetch the FactoryBean instance itself using '&' prefix
            Object factoryBean = context.getBean("&dbConnection");
            System.out.println("FactoryBean Class: " + factoryBean.getClass().getName());
        }
    }
}
```

Expected Output:

```text
=== Blog 25 - FactoryBean Demo ===
[DatabaseConnectionFactoryBean] Constructing complex DatabaseConnection...
[DatabaseConnection] Connected to jdbc:postgresql://localhost:5432/mydb
FactoryBean Class: com.example.springcore.blog25.domain.DatabaseConnectionFactoryBean
```

---

## 5. Key Takeaways

1. `FactoryBean<T>` is a container strategy for encapsulating complex object construction logic.
2. `getBean("name")` returns the created object `T`.
3. `getBean("&name")` returns the `FactoryBean<T>` instance itself using the dereference operator `&`.

---

## 6. Next Blog

**Part 26: `BeanDefinition`**
We dive into `BeanDefinition` metadata internals, convergence across XML/Java/Annotations, and `RootBeanDefinition`.

Where you are: **Blog 25 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility - Complete!).
