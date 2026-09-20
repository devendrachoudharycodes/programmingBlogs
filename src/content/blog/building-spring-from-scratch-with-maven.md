---
title: "Building Spring from Scratch with Maven: Every Line Explained"
description: "Part 4 of the Spring Core series. Step-by-step guide to building a bare Spring project from an empty directory, understanding every element in pom.xml, and bootstrapping XML, Java, and Annotation ApplicationContexts."

pubDate: 2026-09-20
updatedDate: 2026-09-20

author: "Devendra Choudhary"

category: "Spring"
series: "Spring Core"
seriesOrder: 4

tags:
  - java
  - spring
  - spring-core
  - maven
  - build-tools

heroImage: "https://spring.io/img/spring.svg"

draft: false
featured: false

canonicalURL: "https://devendrachoudharycodes.github.io/programmingBlogs/blog/building-spring-from-scratch-with-maven/"
---

| | |
|---|---|
| **Series** | Spring Core Architecture, Part 4 of 49 |
| **Difficulty** | 🟢 Beginner |
| **Reading / Coding / Total** | 25 min / 45 min / ~70 min |
| **Tested with** | Spring Framework 7.0.8, Java 17+, Maven |
| **Concepts gained** | Understanding pom.xml dependencies, transitive artifacts, bootstrapping ApplicationContext 3 ways, Maven project layout |
| **Concepts unlocked** | Part 5 (Using XML, Java Config and Annotations Together), Part 6 (BeanFactory vs ApplicationContext) |

## 1. Prerequisites

**Java concepts required**
- JDK installation, environment variables (`JAVA_HOME`), command-line terminal basics

**Spring concepts required**
- Basic concept of IoC containers and Beans (Parts 1–3)

**Previous blogs required**
- Part 1 (*What Is Spring?*)
- Part 3 (*Beans and the Spring IoC Container*)

**Tooling**
- JDK 17 or newer
- Apache Maven 3.9 or newer

---

## 2. What You Will Learn

- How to structure a standard Maven project for Spring Framework from scratch
- What every single XML line in `pom.xml` does
- Why `spring-context` is the only direct dependency needed for Spring Core
- How transitive dependencies bring in `spring-core`, `spring-beans`, `spring-aop`, and `spring-expression`
- How to bootstrap an `ApplicationContext` in three distinct ways
- The differences between `ClassPathXmlApplicationContext` and `AnnotationConfigApplicationContext`

---

## 3. Why This Topic Matters

Most modern Java tutorials ask you to visit `start.spring.io` and download a pre-packaged Spring Boot project. While convenient, Spring Boot hides the underlying Maven dependencies and container initialization logic behind starter parent POMs and auto-configuration.

By building a pure Spring Framework project from scratch, you see exactly how `spring-context` connects to Java 17 and Maven, without hidden magic.

---

## 4. Building the Project Step-by-Step

### 4.1 Project Directory Structure

Create standard Maven directory layout:

```text
spring-core-learning/
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/example/springcore/blog04/
        │       ├── HelloWorldBean.java
        │       ├── JavaAppConfig.java
        │       └── App.java
        └── resources/
            └── blog04/
                └── beans.xml
```

---

### 4.2 Deconstructing `pom.xml` Line-by-Line

**File:** `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>spring-core-learning</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring.version>7.0.8</spring.version>
    </properties>

    <dependencies>
        <!-- The core container dependency -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>3.5.0</version>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Line Explanations:
1. `<modelVersion>4.0.0</modelVersion>`: Standard Maven POM schema version.
2. `<groupId>`, `<artifactId>`, `<version>`: Unique coordinate identifying your project.
3. `<maven.compiler.release>17</maven.compiler.release>`: Configures javac to target Java 17 release baseline required by Spring Framework 7+.
4. `<artifactId>spring-context</artifactId>`: Pulls in the complete Spring Core IoC container.
5. Transitive Dependencies pulled automatically by `spring-context`:
   - `spring-core`: Fundamental utilities and byte manipulation.
   - `spring-beans`: `BeanFactory` and `BeanDefinition` infrastructure.
   - `spring-aop`: Aspect-oriented proxy support.
   - `spring-expression`: Spring Expression Language (SpEL).

---

## 5. Bootstrapping Container 3 Ways

**File:** `src/main/java/com/example/springcore/blog04/HelloWorldBean.java`

```java
package com.example.springcore.blog04;

public class HelloWorldBean {
    public void sayHello() {
        System.out.println("Hello from Spring Core!");
    }
}
```

### Way 1: XML ApplicationContext

**File:** `src/main/resources/blog04/beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloWorld" class="com.example.springcore.blog04.HelloWorldBean"/>

</beans>
```

### Way 2: Java Configuration ApplicationContext

**File:** `src/main/java/com/example/springcore/blog04/JavaAppConfig.java`

```java
package com.example.springcore.blog04;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JavaAppConfig {

    @Bean
    public HelloWorldBean helloWorld() {
        return new HelloWorldBean();
    }
}
```

### Way 3: Complete Execution App

**File:** `src/main/java/com/example/springcore/blog04/App.java`

```java
package com.example.springcore.blog04;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {

    public static void main(String[] args) {
        System.out.println("=== 1. Bootstrapping with XML ===");
        try (ClassPathXmlApplicationContext xmlContext =
                     new ClassPathXmlApplicationContext("blog04/beans.xml")) {
            HelloWorldBean bean = xmlContext.getBean("helloWorld", HelloWorldBean.class);
            bean.sayHello();
        }

        System.out.println("\n=== 2. Bootstrapping with Java Config ===");
        try (AnnotationConfigApplicationContext javaContext =
                     new AnnotationConfigApplicationContext(JavaAppConfig.class)) {
            HelloWorldBean bean = javaContext.getBean("helloWorld", HelloWorldBean.class);
            bean.sayHello();
        }
    }
}
```

---

## 6. Running with Maven Exec Plugin

Execute from project root:

```bash
mvn compile exec:java -Dexec.mainClass=com.example.springcore.blog04.App
```

Expected Output:

```text
=== 1. Bootstrapping with XML ===
Hello from Spring Core!

=== 2. Bootstrapping with Java Config ===
Hello from Spring Core!
```

---

## 7. Key Takeaways

1. `spring-context` is the single top-level dependency required for Spring Core.
2. `ClassPathXmlApplicationContext` reads XML descriptors from `src/main/resources`.
3. `AnnotationConfigApplicationContext` reads `@Configuration` classes or package paths.
4. Java 17 is the mandatory compiler baseline for Spring Framework 7.0+.

---

## 8. Next Blog

**Part 5: Using XML, Java Configuration and Annotations Together**
We explore hybrid configurations where XML, `@Configuration` classes, and `@ComponentScan` coexist within a single unified `ApplicationContext`.

Where you are: **Blog 4 of 49** (Phase 1: IoC and Container Fundamentals).
