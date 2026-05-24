# 06 — Spring Boot Application

## Project Details

| Detail | Value |
|--------|-------|
| **Framework** | Spring Boot 3.2.5 |
| **Java Version** | 17 |
| **Build Tool** | Maven 3.9.6 |
| **Server Port** | 8081 |

---

## Project Structure

```
springboot-app/
├── Dockerfile
├── Jenkinsfile
├── pom.xml
└── src/
    └── main/
        ├── java/com/example/demo/
        │   ├── DemoApplication.java
        │   └── AppController.java
        └── resources/
            └── application.properties
```

---

## REST Endpoints

| Endpoint | Method | Response |
|----------|--------|----------|
| `/hello` | GET | `Hello, World!` |
| `/heartbeat` | GET | `OK` |

---

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## DemoApplication.java

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

---

## AppController.java

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AppController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }

    @GetMapping("/heartbeat")
    public String heartbeat() {
        return "OK";
    }
}
```

---

## application.properties

```properties
spring.application.name=demo
server.port=8081
```

> 💡 Port **8081** used to avoid conflict with Jenkins on **8080**.

---

## Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

| Line | Purpose |
|------|---------|
| `FROM eclipse-temurin:17-jre-alpine` | Lightweight Java 17 base image |
| `WORKDIR /app` | Set working directory |
| `COPY target/*.jar app.jar` | Copy built JAR from Maven |
| `EXPOSE 8081` | Expose app port |
| `ENTRYPOINT` | Run the JAR |

---

## Build & Run Locally

```bash
cd springboot-app

# Build JAR
mvn clean package -DskipTests

# Run with Maven
mvn spring-boot:run

# OR build & run Docker image
docker build -t springboot-demo .
docker run -p 8081:8081 springboot-demo
```

Test endpoints:
```bash
curl http://localhost:8081/hello
curl http://localhost:8081/heartbeat
```
