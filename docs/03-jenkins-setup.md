# 03 — Jenkins Setup

## Install Jenkins

### Step 1: Add Jenkins GPG Key

```bash
# Fetch key directly from Ubuntu keyserver
sudo gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 7198F4B714ABFC68
sudo gpg --export 7198F4B714ABFC68 | sudo tee /usr/share/keyrings/jenkins-keyring.gpg > /dev/null
```

> ⚠️ Do NOT use `curl | tee` for the key — it causes GPG verification errors.
> Always use `gpg --keyserver` method above.

### Step 2: Add Jenkins Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.gpg] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Step 3: Install Jenkins

```bash
sudo apt-get update
sudo apt-get install jenkins -y
```

> **Version Installed:** Jenkins 2.555.2

### Step 4: Start Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

---

## Access Jenkins UI

```
http://65.1.134.229:8080
```

### Get Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### First Login Steps

1. Paste the admin password
2. Click **"Install Suggested Plugins"** and wait
3. Create **Admin User**
4. Set Jenkins URL → **Save and Finish**
5. Click **"Start using Jenkins"**

---

## Install Additional Plugins

**Manage Jenkins → Plugins → Available Plugins**

Search and install:
- ✅ **Docker Pipeline**
- ✅ **SonarQube Scanner**

---

## Configure SonarQube Server in Jenkins

**Manage Jenkins → System → SonarQube Servers**

| Field | Value |
|-------|-------|
| ✅ Enable | Check "Environment variables" |
| **Name** | `sonarqube` |
| **Server URL** | `http://65.1.134.229:9000` |
| **Authentication Token** | Select `sonarqube` credential |

---

## Configure SonarQube Scanner Tool

**Manage Jenkins → Tools → SonarQube Scanner**

| Field | Value |
|-------|-------|
| **Name** | `sonar-scanner` |
| **Install automatically** | ✅ Checked |

---

## Add SonarQube Token as Credential

**Manage Jenkins → Credentials → Global → Add Credentials**

| Field | Value |
|-------|-------|
| **Kind** | Secret text |
| **Secret** | *(paste SonarQube token)* |
| **ID** | `sonarqube` |
| **Description** | SonarQube Auth Token |

---

## Configure Pipeline Job

1. **New Item** → Enter name → Select **Pipeline**
2. Scroll to **Pipeline** section
3. **Definition** → `Pipeline script from SCM`
4. **SCM** → `Git`
5. **Repository URL** → `https://github.com/adikhan14/jenkins-springboot-demo.git`
6. **Branch** → `*/main`
7. **Script Path** → `springboot-app/Jenkinsfile`
8. Click **Save**

---

## Common Errors

### Jenkins fails to start
```
Running with Java 17 — older than minimum required version (Java 21)
```
**Fix:** `sudo apt install openjdk-21-jre -y` then restart Jenkins.

### GPG key error during install
```
NO_PUBKEY 7198F4B714ABFC68
```
**Fix:** Use `gpg --keyserver` method instead of `curl | tee`.
