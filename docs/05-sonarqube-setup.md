# 05 — SonarQube Setup

## Create SonarQube User

SonarQube cannot run as root — must use a dedicated user.

```bash
sudo su -
adduser sonarqube
# Fill in details when prompted
# Full Name: Sonar Qube
```

---

## Install unzip (as ubuntu user)

```bash
exit          # back to ubuntu
sudo apt update && sudo apt install unzip -y
```

---

## Download & Extract SonarQube

```bash
# Switch to sonarqube user
sudo su - sonarqube

# Download SonarQube 10.4.1
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip

# Extract
unzip sonarqube-10.4.1.88267.zip

# Remove zip to save disk space (important!)
rm sonarqube-10.4.1.88267.zip
```

> 💡 `~` for sonarqube user = `/home/sonarqube`
> So files are at `/home/sonarqube/sonarqube-10.4.1.88267`

---

## Set Permissions

```bash
exit    # back to ubuntu

sudo chmod -R 755 /home/sonarqube/sonarqube-10.4.1.88267
sudo chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267
```

### Why these commands?

| Command | Purpose |
|---------|---------|
| `chown -R sonarqube:sonarqube` | Makes sonarqube user the owner |
| `chmod -R 755` | Owner: full access, Others: read+execute |

---

## Fix Java 21 Security Manager Issue

SonarQube 10.4.1 uses Java Security Manager which throws `UnsupportedOperationException` in Java 18+.

```bash
sudo su - sonarqube

# Fix for Web Server
echo 'sonar.web.javaAdditionalOpts=-Djava.security.manager=allow' >> ~/sonarqube-10.4.1.88267/conf/sonar.properties

# Fix for Compute Engine
echo 'sonar.ce.javaAdditionalOpts=-Djava.security.manager=allow' >> ~/sonarqube-10.4.1.88267/conf/sonar.properties

# Verify both lines added
tail -5 ~/sonarqube-10.4.1.88267/conf/sonar.properties
```

> ⚠️ Both `sonar.web` AND `sonar.ce` need this fix — missing either one will cause a crash.

---

## Start SonarQube

```bash
# As sonarqube user
~/sonarqube-10.4.1.88267/bin/linux-x86-64/sonar.sh start

# Wait 1-2 minutes then check
~/sonarqube-10.4.1.88267/bin/linux-x86-64/sonar.sh status
```

Expected:
```
SonarQube is running (PID)
```

### Run in Console Mode (for debugging)
```bash
~/sonarqube-10.4.1.88267/bin/linux-x86-64/sonar.sh console
```

---

## Access SonarQube UI

```
http://65.1.134.229:9000
```

> Default credentials: **admin / admin**
> You will be prompted to change password on first login.

---

## Generate Token for Jenkins

1. Login to SonarQube
2. **My Account → Security → Generate Token**
3. Name: `jenkins-token`
4. Type: **Global Analysis Token**
5. Click **Generate** and **copy the token**

---

## SonarQube Logs Location

| Log File | Purpose |
|----------|---------|
| `~/sonarqube-10.4.1.88267/logs/sonar.log` | Main application log |
| `~/sonarqube-10.4.1.88267/logs/web.log` | Web server log |
| `~/sonarqube-10.4.1.88267/logs/es.log` | Elasticsearch log |
| `~/sonarqube-10.4.1.88267/logs/ce.log` | Compute Engine log |

---

## Common Errors

### Web Server crashes immediately (web.log empty)
```
Exception: java.lang.UnsupportedOperationException: The Security Manager is deprecated
```
**Fix:** Add `-Djava.security.manager=allow` to both `sonar.web.javaAdditionalOpts` and `sonar.ce.javaAdditionalOpts` in `sonar.properties`.

### sonarqube user can't run sudo
```
sudo: I'm sorry sonarqube. I'm afraid I can't do that
```
**Fix:** This is expected. Switch to `ubuntu` user for `sudo` commands, then switch back to `sonarqube`.

### Compute Engine crashes
Add `sonar.ce.javaAdditionalOpts=-Djava.security.manager=allow` — both web AND ce need the fix.
