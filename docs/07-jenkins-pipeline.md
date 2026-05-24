# 07 — Jenkins Pipeline

## Jenkinsfile

The `Jenkinsfile` defines the CI/CD pipeline. It can be provided in two ways:

### Option 1 — Pipeline Script from SCM (Recommended)
Store the `Jenkinsfile` in your GitHub repository and let Jenkins pull it automatically on every build.

**Configure in Jenkins UI:**
1. **New Item** → name it → select **Pipeline**
2. **Pipeline section:**
   - Definition: `Pipeline script from SCM`
   - SCM: `Git`
   - Repository URL: `<your-repo-url>`
   - Branch: `*/main`
   - Script Path: `springboot-app/Jenkinsfile`
3. **Save** → **Build Now**

> 💡 Every time you push changes to GitHub, Jenkins picks up the latest `Jenkinsfile` automatically.

---

### Option 2 — Pipeline Script directly in Jenkins
Write the pipeline script directly in the Jenkins UI — no Git repo needed.

**Configure in Jenkins UI:**
1. **New Item** → name it → select **Pipeline**
2. **Pipeline section:**
   - Definition: `Pipeline script`
   - Paste the Jenkinsfile content directly in the text box
3. **Save** → **Build Now**

> ⚠️ Changes to the pipeline won't be version controlled. Use Option 1 for real projects.

---

## Jenkinsfile Content

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    stages {

        stage('Build and Test') {
            steps {
                sh 'cd springboot-app && mvn clean package -DskipTests=false'
            }
        }

        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://<your-ec2-ip>:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        cd springboot-app && mvn sonar:sonar \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.host.url=${SONAR_URL}
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            environment {
                DOCKER_IMAGE = "springboot-demo:${BUILD_NUMBER}"
            }
            steps {
                sh '''
                    apt-get update -qq && apt-get install -y -qq docker.io
                    cd springboot-app && docker build -t ${DOCKER_IMAGE} .
                '''
                echo "Docker image built: springboot-demo:${BUILD_NUMBER}"
            }
        }

    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
        }
    }
}
```

---

## Pipeline Stages Explained

### Stage 1 — Build and Test

| What | Detail |
|------|--------|
| `clean` | Removes previous build artifacts |
| `package` | Compiles code and creates JAR |
| `-DskipTests=false` | Runs unit tests |
| **Output** | `target/demo-0.0.1-SNAPSHOT.jar` |

---

### Stage 2 — Static Code Analysis

| What | Detail |
|------|--------|
| `sonar:sonar` | Runs SonarQube analysis |
| `SONAR_AUTH_TOKEN` | Pulled from Jenkins credentials |
| `SONAR_URL` | Points to your SonarQube instance |
| **Output** | Code quality report in SonarQube UI |

---

### Stage 3 — Build Docker Image

| What | Detail |
|------|--------|
| `apt-get install docker.io` | Installs Docker CLI inside Maven container |
| `docker build` | Builds image using host Docker daemon via socket |
| `${BUILD_NUMBER}` | Jenkins build number used as image tag |
| **Output** | Docker image available on EC2 host |

---

## How the Docker Agent Works

```
Jenkins Server (EC2 — Java 21)
    │
    ├── pulls image: maven:3.9.6-eclipse-temurin-17
    │
    └── runs container:
            --user root
            -v /var/run/docker.sock:/var/run/docker.sock
            │
            ├── Stage 1: mvn clean package       ✅ (Java 17 inside container)
            ├── Stage 2: mvn sonar:sonar          ✅ (talks to SonarQube)
            └── Stage 3: docker build             ✅ (CLI installed, socket mounted)
```

## What `agent` Does

| Agent Type | Where Pipeline Runs |
|-----------|---------------------|
| `agent any` | Directly on Jenkins EC2 host |
| `agent { docker { image '...' } }` | Inside a Docker container |

## Docker Socket Explained

| Part | Meaning |
|------|---------|
| `--user root` | Run container as root (needed to install packages) |
| `-v /var/run/docker.sock:/var/run/docker.sock` | Mount host Docker socket into container |

> The socket connects the container to the host's Docker daemon.
> The container still needs Docker **CLI** (`docker.io`) to communicate with it.
