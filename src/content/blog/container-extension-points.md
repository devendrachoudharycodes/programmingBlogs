---
title: "Container Extension Points: SPIs for Customizing Spring"
description: "Part 23 of the Spring Core series. Deep architectural comparison of BeanFactoryPostProcessor, BeanDefinitionRegistryPostProcessor, BeanPostProcessor, and ApplicationListener."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 23

tags:
  - java
  - spring
  - spring-core
  - extension-points
  - container-spi

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/container-extension-points/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 23 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 45 min / 60 min / ~105 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Extension point hierarchy, BeanFactoryPostProcessor, BeanDefinitionRegistryPostProcessor, BeanPostProcessor, ApplicationListener |
| **Concepts unlocked** | Part 24 (How Spring AOP Uses BPP), Part 27 (Programmatic Registration) |

## 1. Prerequisites

**Spring concepts required**
- `BeanDefinition` metadata and `BeanPostProcessor` execution (Parts 3 & 22)

**Previous blogs required**
- Part 3 (*Beans and the Spring IoC Container*)
- Part 22 (*BeanPostProcessor*)

---

## 2. Extension Point Execution Order

Spring processes container extension points in a strict sequence:

```text
1. BeanDefinition Metadata Loaded (XML / Java Config / Scanning)
   ↓
2. BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry()
   (Adds new BeanDefinitions programmatically)
   ↓
3. BeanFactoryPostProcessor.postProcessBeanFactory()
   (Modifies existing BeanDefinition metadata recipes)
   ↓
4. Bean Instantiation Phase Begins
   ↓
5. BeanPostProcessor.postProcessBeforeInitialization()
   ↓
6. @PostConstruct / InitializingBean Callbacks
   ↓
7. BeanPostProcessor.postProcessAfterInitialization() (Proxies created)
```

---

## 3. SPI Interface Comparison

| Extension Interface | Target Subject | When Executed | Typical Use Case |
|---|---|---|---|
| **`BeanDefinitionRegistryPostProcessor`** | `BeanDefinitionRegistry` | Before `BeanFactoryPostProcessor` | Dynamically register new bean definitions programmatically. |
| **`BeanFactoryPostProcessor`** | `ConfigurableListableBeanFactory` | After registry, before bean creation | Modify bean metadata (e.g. resolve `${db.url}` placeholders). |
| **`BeanPostProcessor`** | Bean Instances | During bean initialization phase | Wrap beans in proxies (AOP, security, transaction decorators). |
| **`ApplicationListener`** | `ApplicationEvent` | On event publication | Respond to container events (`ContextRefreshedEvent`). |

---

## 4. Complete Working Example

**File:** `src/main/java/com/example/springcore/blog23/domain/CustomBfpp.java`

```java
package com.example.springcore.blog23.domain;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomBfpp implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("[BeanFactoryPostProcessor] Executed! Total Bean Definitions registered: "
                + beanFactory.getBeanDefinitionCount());
    }
}
```

**File:** `src/main/java/com/example/springcore/blog23/ExtensionDemoApp.java`

```java
package com.example.springcore.blog23;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class ExtensionDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog23.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 23 - Container Extension Points Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            System.out.println("Context startup complete!");
        }
    }
}
```

Expected Output:

```text
=== Blog 23 - Container Extension Points Demo ===
[BeanFactoryPostProcessor] Executed! Total Bean Definitions registered: 6
Context startup complete!
```

---

## 5. Key Takeaways

1. `BeanFactoryPostProcessor` runs on **metadata** before any beans are created.
2. `BeanPostProcessor` runs on **instances** during bean creation.
3. `@Bean` methods defining `BeanFactoryPostProcessor` MUST be declared `static`.

---

## 6. Next Blog

**Part 24: How Spring AOP Uses `BeanPostProcessor`**
We explore how Spring AOP leverages `AnnotationAwareAspectJAutoProxyCreator` (a `BeanPostProcessor`) to wrap target beans in CGLIB or JDK dynamic proxies.

Where you are: **Blog 23 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
