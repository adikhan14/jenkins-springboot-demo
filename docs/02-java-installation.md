# 02 — Java Installation

## Why Two Java Versions?

| Java Version | Used For | Required By |
|-------------|----------|-------------|
| **Java 17** | Spring Boot App build | `pom.xml` |
| **Java 21** | Jenkins & SonarQube runtime | Jenkins 2.555.2+ |

---

## Install Java 17 & Java 21

```bash
sudo apt update

# Install Java 17
sudo apt install openjdk-17-jre -y

# Install Java 21
sudo apt install openjdk-21-jre -y
```

---

## Verify Installation

```bash
java -version
```

Expected output:
```
openjdk version "17.0.18" 2026-01-20
OpenJDK Runtime Environment (build 17.0.18+8-Ubuntu-1)
OpenJDK 64-Bit Server VM (build 17.0.18+8-Ubuntu-1, mixed mode, sharing)
```

---

## Java Locations

| Version | Path |
|---------|------|
| Java 17 | `/usr/lib/jvm/java-17-openjdk-amd64` |
| Java 21 | `/usr/lib/jvm/java-21-openjdk-amd64` |

---

## Common Errors

### Jenkins won't start with Java 17
```
Running with Java 17... which is older than the minimum required version (Java 21).
Supported Java versions are: [21, 25]
```
**Fix:** Install Java 21 — `sudo apt install openjdk-21-jre -y`
