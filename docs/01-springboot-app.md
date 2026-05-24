# 01 — Spring Boot App & GitHub

## Create Spring Boot Project

1. Go to [start.spring.io](https://start.spring.io)
2. Configure:

| Setting | Value |
|---------|-------|
| **Project** | Maven |
| **Language** | Java |
| **Spring Boot** | 3.2.5 |
| **Java** | 17 |
| **Dependency** | Spring Web |

3. Generate and extract the project

---

## REST Endpoints

Add two endpoints in `AppController.java`:

```java
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

| Endpoint | Response |
|----------|----------|
| `/hello` | `Hello, World!` |
| `/heartbeat` | `OK` |

---

## application.properties

```properties
spring.application.name=demo
server.port=8081
```

---

## Dockerfile

Create a `Dockerfile` in the project root:

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## Push to GitHub

```bash
git init
git add .
git commit -m "initial commit"
gh repo create jenkins-springboot-demo --public --source=. --remote=origin --push
```

> 💡 Uses GitHub CLI (`gh`). Make sure you're logged in with `gh auth login` first.
