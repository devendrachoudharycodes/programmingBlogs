---
title: "How Spring AOP Uses BeanPostProcessor: Proxy Creation and Interception"
description: "Part 24 of the Spring Core series. How Spring AOP leverages BeanPostProcessor to wrap target beans into JDK Dynamic Proxies or CGLIB subclasses."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 24

tags:
  - java
  - spring
  - spring-core
  - aop
  - dynamic-proxies

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/how-spring-aop-uses-beanpostprocessor/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 24 of 49 |
| **Difficulty** | 🔴 Advanced |
| **Reading / Coding / Total** | 40 min / 55 min / ~95 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | IoC and AOP bridge, JDK Dynamic Proxy vs CGLIB, BeanPostProcessor proxy wrapping, method interception |
| **Concepts unlocked** | Part 25 (FactoryBean), Part 29 (@Configuration and @Bean Semantics) |

## 1. Prerequisites

**Spring concepts required**
- `BeanPostProcessor` lifecycle hooks (Part 22)

**Previous blogs required**
- Part 22 (*BeanPostProcessor*)
- Part 23 (*Container Extension Points*)

---

## 2. What You Will Learn

- How Spring AOP bridges with Spring IoC via `BeanPostProcessor`
- The Proxy Creation Sequence: `Original Target ──► BPP.postProcessAfterInitialization() ──► Proxy Wrapper`
- Differences between JDK Dynamic Proxies (interface-based) and CGLIB Proxies (subclass-based)
- Why calling `context.getBean()` for an AOP-advised bean returns a Proxy instance, not the target class
- Building a custom proxy-creating `BeanPostProcessor` using Java's `java.lang.reflect.Proxy`

---

## 3. Proxy Creation Sequence

```text
1. Target Bean Instantiated & Initialized
   ↓
2. BeanPostProcessor.postProcessAfterInitialization(targetBean, beanName)
   ↓
3. Is Target Bean Advised by AOP Aspects?
   ├── YES ──► Construct Dynamic Proxy Wrapper (JDK or CGLIB) ──► Return PROXY
   └── NO  ──► Return Original Target Bean
   ↓
4. ApplicationContext Stores & Exposes Proxy Instance
```

---

## 4. Complete Working Example (Custom Proxy BPP)

**File:** `src/main/java/com/example/springcore/blog24/domain/PaymentService.java`

```java
package com.example.springcore.blog24.domain;

public interface PaymentService {
    void processPayment(String orderId);
}
```

**File:** `src/main/java/com/example/springcore/blog24/domain/PaymentServiceImpl.java`

```java
package com.example.springcore.blog24.domain;

import org.springframework.stereotype.Component;

@Component
public class PaymentServiceImpl implements PaymentService {

    @Override
    public void processPayment(String orderId) {
        System.out.println("[PaymentServiceImpl] Executing payment for " + orderId);
    }
}
```

**File:** `src/main/java/com/example/springcore/blog24/domain/SimpleTimingProxyBpp.java`

```java
package com.example.springcore.blog24.domain;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

import java.lang.reflect.Proxy;

@Component
public class SimpleTimingProxyBpp implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof PaymentService) {
            System.out.println("[Proxy BPP] Wrapping " + beanName + " in JDK Dynamic Proxy...");

            return Proxy.newProxyInstance(
                    bean.getClass().getClassLoader(),
                    bean.getClass().getInterfaces(),
                    (proxy, method, args) -> {
                        long start = System.currentTimeMillis();
                        System.out.println("[Proxy Interceptor] BEFORE method execution: " + method.getName());

                        Object result = method.invoke(bean, args);

                        long duration = System.currentTimeMillis() - start;
                        System.out.println("[Proxy Interceptor] AFTER method execution (" + duration + "ms)");
                        return result;
                    }
            );
        }
        return bean;
    }
}
```

**File:** `src/main/java/com/example/springcore/blog24/AopProxyDemoApp.java`

```java
package com.example.springcore.blog24;

import com.example.springcore.blog24.domain.PaymentService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

public class AopProxyDemoApp {

    @Configuration
    @ComponentScan(basePackages = "com.example.springcore.blog24.domain")
    static class Config {}

    public static void main(String[] args) {
        System.out.println("=== Blog 24 - How Spring AOP Uses BPP Demo ===");

        try (AnnotationConfigApplicationContext context =
                     new AnnotationConfigApplicationContext(Config.class)) {

            PaymentService service = context.getBean(PaymentService.class);
            System.out.println("Returned Bean Class: " + service.getClass().getName());

            service.processPayment("ORD-777");
        }
    }
}
```

Expected Output:

```text
=== Blog 24 - How Spring AOP Uses BPP Demo ===
[Proxy BPP] Wrapping paymentServiceImpl in JDK Dynamic Proxy...
Returned Bean Class: jdk.proxy2.$Proxy12
[Proxy Interceptor] BEFORE method execution: processPayment
[PaymentServiceImpl] Executing payment for ORD-777
[Proxy Interceptor] AFTER method execution (0ms)
```

---

## 5. Key Takeaways

1. Spring AOP integrates with IoC via `BeanPostProcessor.postProcessAfterInitialization()`.
2. Advised beans returned by `getBean()` are dynamic proxies, not raw target instances.
3. Method calls on proxied beans pass through interception logic before delegating to the target object.

---

## 6. Next Blog

**Part 25: `FactoryBean`**
We explore `FactoryBean`, creating complex or third-party objects with custom factory logic, and the `&beanName` syntax.

Where you are: **Blog 24 of 49** (Phase 3: Scopes, Lifecycle, and Extensibility).
