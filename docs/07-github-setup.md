# 07 — GitHub Setup

## Initialize Git Repository

```bash
cd "/Users/apple/Jenkins Pipeline"
git init
```

---

## Add .gitignore

```bash
# Root .gitignore — exclude sensitive files
echo "*.pem" >> .gitignore
```

> ⚠️ NEVER commit `.pem` files to GitHub — bots scan public repos for keys within seconds!

---

## First Commit

```bash
git add .
git commit -m "initial commit"
```

---

## Remove PEM File If Accidentally Committed

```bash
# Remove from git tracking (keeps file on disk)
git rm --cached aws-pem-file.pem

# Add to gitignore
echo "*.pem" >> .gitignore
git add .gitignore

# Amend commit to remove it
git commit --amend -m "initial commit"

# Verify it's gone
git ls-files | grep pem   # should return nothing
```

---

## Create GitHub Repo via CLI

```bash
# Using GitHub CLI (gh)
gh repo create jenkins-springboot-demo --public --source=. --remote=origin --push
```

**Result:**
```
https://github.com/adikhan14/jenkins-springboot-demo
```

---

## Push Changes

```bash
git add .
git commit -m "your message"
git push origin main
```

---

## Repository Details

| Detail | Value |
|--------|-------|
| **Repo URL** | https://github.com/adikhan14/jenkins-springboot-demo |
| **Owner** | adikhan14 |
| **Branch** | main |
| **Visibility** | Public |

---

## Files in Repository

```
jenkins-springboot-demo/
├── .gitignore
└── springboot-app/
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
