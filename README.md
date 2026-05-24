# Jenkins CI/CD Pipeline on AWS EC2

A complete CI/CD pipeline setup using Jenkins, SonarQube, Docker, and Spring Boot on AWS EC2.

---

## Architecture

```
GitHub Repo
    ↓ triggers
Jenkins (EC2 :8080)
    ↓ runs pipeline inside
Maven Docker Container
    ├── Stage 1: Build & Test (Maven + Java 17)
    ├── Stage 2: Static Code Analysis (SonarQube :9000)
    └── Stage 3: Build Docker Image (springboot-demo)
```

---

## Documentation

| # | Guide | Description |
|---|-------|-------------|
| 1 | [EC2 Setup](docs/01-ec2-setup.md) | Launch EC2 instance, security groups, SSH |
| 2 | [Java Installation](docs/02-java-installation.md) | Install Java 17 & 21 |
| 3 | [Jenkins Setup](docs/03-jenkins-setup.md) | Install, configure & access Jenkins |
| 4 | [Docker Setup](docs/04-docker-setup.md) | Install Docker, fix permissions |
| 5 | [SonarQube Setup](docs/05-sonarqube-setup.md) | Install, configure & access SonarQube |
| 6 | [Spring Boot App](docs/06-springboot-app.md) | App code, endpoints, Dockerfile |
| 7 | [GitHub Setup](docs/07-github-setup.md) | Create repo, push code |
| 8 | [Jenkins Pipeline](docs/08-jenkins-pipeline.md) | Jenkinsfile, pipeline stages |
| 9 | [Troubleshooting](docs/09-troubleshooting.md) | All errors & fixes encountered |

---

## Quick Reference

| Service | Port | URL |
|---------|------|-----|
| Jenkins | 8080 | http://65.1.134.229:8080 |
| SonarQube | 9000 | http://65.1.134.229:9000 |
| Spring Boot | 8081 | http://65.1.134.229:8081 |

---

## GitHub Repository

```
https://github.com/adikhan14/jenkins-springboot-demo
```

## Tech Stack

| Tool | Version | Purpose |
|------|---------|---------|
| Jenkins | 2.555.2 | CI/CD Server |
| SonarQube | 10.4.1 | Static Code Analysis |
| Docker | latest | Containerization |
| Spring Boot | 3.2.5 | Application Framework |
| Java | 17 (app) / 21 (Jenkins) | Runtime |
| Maven | 3.9.6 | Build Tool |
| Ubuntu | Latest | EC2 OS |
