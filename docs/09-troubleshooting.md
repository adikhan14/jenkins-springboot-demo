# 09 — Troubleshooting

All errors encountered during setup and their fixes.

---

## EC2 / SSH

### SSH Connection Timeout
```
ssh: connect to host 13.126.6.204 port 22: Operation timed out
```
**Cause:** Security Group doesn't allow inbound SSH.
**Fix:** AWS Console → Security Group → Add inbound rule: TCP port 22.

---

## Jenkins Installation

### GPG Key Verification Failed
```
NO_PUBKEY 7198F4B714ABFC68
E: The repository 'https://pkg.jenkins.io/debian binary/ Release' is not signed.
E: Package 'jenkins' has no installation candidate
```
**Cause:** Used `curl | tee` to add key — doesn't work on newer Ubuntu.
**Fix:**
```bash
sudo gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7198F4B714ABFC68
sudo gpg --export 7198F4B714ABFC68 | sudo tee /usr/share/keyrings/jenkins-keyring.gpg > /dev/null
```

---

### Jenkins Won't Start — Wrong Java Version
```
Running with Java 17 from /usr/lib/jvm/java-17-openjdk-amd64,
which is older than the minimum required version (Java 21).
Supported Java versions are: [21, 25]
```
**Cause:** Jenkins 2.555.2 requires Java 21 minimum.
**Fix:**
```bash
sudo apt install openjdk-21-jre -y
sudo systemctl restart jenkins
```

---

### Jenkins Service Didn't Auto-Start After Install
```
Could not execute systemctl: at /usr/bin/deb-systemd-invoke line 148.
```
**Cause:** Normal behavior on fresh EC2 — `deb-systemd-invoke` can't communicate mid-install.
**Fix:** Start manually after install:
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Docker

### Permission Denied on Docker Socket
```
permission denied while trying to connect to the Docker daemon socket
at unix:///var/run/docker.sock
```
**Cause:** Jenkins user not in `docker` group.
**Fix:**
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### Docker CLI Not Found in Pipeline
```
docker: not found
/var/lib/jenkins/workspace/...: 1: docker: not found
```
**Cause:** Maven container doesn't have Docker CLI installed.
**Fix:** Add to Build Docker Image stage:
```groovy
sh 'apt-get update -qq && apt-get install -y -qq docker.io'
```

---

## Disk Space

### No Space Left on Device
```
write libcrypto.so.3: no space left on device
java.io.IOException: No space left on device
```
**Cause:** 8GB root volume filled up by Jenkins + Docker images + SonarQube.
**Fix:**
```bash
# Free up space immediately
sudo rm /home/sonarqube/sonarqube-10.4.1.88267.zip
sudo docker system prune -a -f
sudo apt-get clean
df -h
```
**Long-term fix:** Expand EBS volume to 20GB from AWS Console, then:
```bash
sudo growpart /dev/xvda 1
sudo resize2fs /dev/root
```

---

## SonarQube

### Web Server Crashes — Security Manager Error
```
Exception in thread "main" java.lang.UnsupportedOperationException:
The Security Manager is deprecated and will be removed in a future release
    at java.base/java.lang.System.setSecurityManager(System.java:431)
```
**Cause:** SonarQube 10.4.1 uses Java Security Manager, blocked by default in Java 18+.
**Fix:**
```bash
echo 'sonar.web.javaAdditionalOpts=-Djava.security.manager=allow' >> ~/sonarqube-10.4.1.88267/conf/sonar.properties
echo 'sonar.ce.javaAdditionalOpts=-Djava.security.manager=allow' >> ~/sonarqube-10.4.1.88267/conf/sonar.properties
```
> ⚠️ Both `web` AND `ce` need this fix!

### Compute Engine Crashes After Web Server Starts
```
Process exited with exit value [Compute Engine]: 1
```
**Cause:** Same Security Manager issue but for Compute Engine.
**Fix:** Add `sonar.ce.javaAdditionalOpts=-Djava.security.manager=allow` to sonar.properties.

### sonarqube User Can't Use sudo
```
sudo: I'm sorry sonarqube. I'm afraid I can't do that
```
**Cause:** `sonarqube` is a restricted user with no sudo privileges — this is intentional.
**Fix:** Switch to `ubuntu` user for sudo commands, then back to `sonarqube`.

---

## Maven / Build

### Java Release Version Not Supported
```
Fatal error compiling: error: release version 17 not supported
```
**Cause:** Docker agent (`abhishekf5/maven-abhishek-docker-agent:v1`) uses Java 11, which can't compile Java 17 code.
**Fix:** Use `maven:3.9.6-eclipse-temurin-17` as the Docker agent image.

---

## Quick Reference — Error to Fix

| Error | Fix |
|-------|-----|
| SSH timeout | Open port 22 in Security Group |
| GPG key error | Use `gpg --keyserver` method |
| Jenkins needs Java 21 | `sudo apt install openjdk-21-jre -y` |
| Docker permission denied | `sudo usermod -aG docker jenkins` + restart |
| Docker CLI not found | `apt-get install docker.io` in pipeline stage |
| No space left on device | `docker system prune -a -f` + expand EBS to 20GB |
| SonarQube Security Manager | Add `-Djava.security.manager=allow` to sonar.properties |
| Maven release 17 not supported | Use `maven:3.9.6-eclipse-temurin-17` agent |
