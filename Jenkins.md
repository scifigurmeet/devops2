
---

# ⭐ UNIT 6 – CI/CD with Jenkins (Practical Guide)

**Target Audience:** Absolute Beginners
**Platform:** Windows 10/11 + Docker Desktop
**Approach:** Learn Jenkins by **doing**, not theory overload

---

## 🎯 What Students Will Learn

By the end of this unit, students will be able to:

✅ Install Jenkins using Docker Compose
✅ Understand Jenkins Architecture (Controller/Agent)
✅ Create Freestyle & Pipeline Jobs
✅ Write Jenkinsfile (Declarative & Scripted)
✅ Integrate Jenkins with GitHub
✅ Build & push Docker images
✅ Run Maven builds in Jenkins
✅ Trigger pipelines automatically
✅ Understand real CI/CD deployment flows

---

## 🧰 Prerequisites (Must Have)

Make sure the following are installed:

| Tool           | Purpose           |
| -------------- | ----------------- |
| Docker Desktop | Run Jenkins       |
| Git            | Source control    |
| GitHub Account | Repo & Webhooks   |
| Browser        | Access Jenkins UI |

Verify Docker is running:

```bash
docker version
```

---

# 🚀 PART 1: Jenkins Installation using Docker Compose

---

## 📁 Step 1: Create Project Folder

```bash
mkdir jenkins-docker
cd jenkins-docker
```

---

## 📄 Step 2: Create `docker-compose.yml`

```yaml
version: "3.8"

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped

volumes:
  jenkins_home:
```

🧠 **Why Docker socket mount?**
So Jenkins can build Docker images.

---

## ▶️ Step 3: Start Jenkins

```bash
docker compose up -d
```

Check container:

```bash
docker ps
```

---

## 🌐 Step 4: Access Jenkins UI

Open browser:

```
http://localhost:8080
```

---

## 🔐 Step 5: Unlock Jenkins

Get admin password:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Paste it → **Continue**

---

## ⚙️ Step 6: Initial Setup

Choose:
👉 **Install Suggested Plugins**

Create:

* Admin Username
* Password

🎉 Jenkins is ready!

---

# 🏗️ PART 2: Jenkins Foundations

---

## 🧠 Jenkins Architecture (Controller / Agent Model)

| Component           | Role                |
| ------------------- | ------------------- |
| Controller (Master) | UI, scheduling jobs |
| Agent               | Executes jobs       |
| Executors           | Parallel job slots  |

👉 **In Docker setup:** Controller = Agent (same container)

---

## 🖥️ Jenkins UI Overview

| Section        | Purpose        |
| -------------- | -------------- |
| Dashboard      | All jobs       |
| Manage Jenkins | Configurations |
| Build History  | Job logs       |
| Credentials    | Secrets        |

---

## 🔌 Plugin Management

📍 Manage Jenkins → Plugins

Install these (if not already):

* Git
* Pipeline
* Docker Pipeline
* Maven Integration

---

## 🔐 Security: Users & Roles (Basics)

📍 Manage Jenkins → Security

* Authentication: Jenkins own user database
* Authorization: Logged-in users can do anything (for labs)

---

# 🧪 PART 3: Freestyle Job (First CI Job)

---

## 📦 Example: Simple Git Checkout Job

### Step 1: Create Job

* New Item → Freestyle
* Name: `freestyle-demo`

### Step 2: Source Code Management

* Git
* Repo URL:

  ```
  https://github.com/spring-projects/spring-petclinic.git
  ```

### Step 3: Build Step

Add:

```
Execute Shell
```

Command:

```bash
echo "Hello Jenkins"
```

### Step 4: Build

✅ Job runs successfully

---

🧠 **Limitation of Freestyle Jobs**

* Hard to maintain
* No version control
* Not scalable

➡️ **Solution: Jenkins Pipelines**

---

# 🔁 PART 4: Jenkins Pipelines (Core of CI/CD)

---

## ⚔️ Freestyle vs Pipeline

| Feature            | Freestyle | Pipeline |
| ------------------ | --------- | -------- |
| Code based         | ❌         | ✅        |
| Version controlled | ❌         | ✅        |
| Complex flows      | ❌         | ✅        |

---

## 📄 Jenkinsfile Structure

```groovy
pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        echo 'Building...'
      }
    }
  }

  post {
    always {
      echo 'Done'
    }
  }
}
```

---

## 🧱 Declarative Pipeline (Recommended)

### Create Pipeline Job

* New Item → Pipeline
* Name: `pipeline-demo`

### Pipeline Script:

```groovy
pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/spring-projects/spring-petclinic.git'
      }
    }

    stage('Build') {
      steps {
        echo 'Building project'
      }
    }

    stage('Test') {
      steps {
        echo 'Running tests'
      }
    }
  }

  post {
    success {
      echo 'Pipeline Successful'
    }
  }
}
```

---

## 🧠 Scripted Pipeline (Just Concept)

```groovy
node {
  stage('Build') {
    echo 'Build'
  }
}
```

👉 **Use Declarative in teaching**

---

## 🎯 Parameters & Environment Variables

```groovy
pipeline {
  agent any

  parameters {
    string(name: 'ENV', defaultValue: 'dev')
  }

  environment {
    APP_NAME = "myapp"
  }

  stages {
    stage('Print') {
      steps {
        echo "Env: ${params.ENV}"
        echo "App: ${APP_NAME}"
      }
    }
  }
}
```

---

## 🌿 Multibranch Pipelines (Concept)

* Automatically builds:

  * main
  * dev
  * feature/*

Requires:

* Jenkinsfile in repo

---

# 🐳 PART 5: Docker & Jenkins Integration

---

## 📦 Sample Dockerfile

Create file `Dockerfile`:

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

---

## 🧪 Pipeline: Build Docker Image

```groovy
pipeline {
  agent any

  stages {
    stage('Build Image') {
      steps {
        sh 'docker build -t demo-nginx .'
      }
    }
  }
}
```

---

## 🚀 Push Image to Docker Hub

### Step 1: Add Credentials

* Docker Hub Username/Password
* ID: `dockerhub-creds`

### Step 2: Pipeline

```groovy
pipeline {
  agent any

  stages {
    stage('Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
        }
      }
    }

    stage('Build & Push') {
      steps {
        sh '''
          docker build -t username/demo-app .
          docker push username/demo-app
        '''
      }
    }
  }
}
```

---

# ☕ PART 6: Jenkins & Maven Integration

---

## ⚙️ Install Maven in Jenkins

📍 Manage Jenkins → Global Tool Configuration
Add:

* Maven Name: `maven3`
* Install automatically

---

## 🧪 Maven Pipeline Example

```groovy
pipeline {
  agent any

  tools {
    maven 'maven3'
  }

  stages {
    stage('Build') {
      steps {
        git 'https://github.com/spring-projects/spring-petclinic.git'
        sh 'mvn clean package'
      }
    }
  }
}
```

---

## 📊 Test Reports & Coverage (Concept)

* Surefire reports
* JaCoCo plugin

(Beyond beginner labs)

---

# 🚦 PART 7: CI/CD Deployment Flows

---

## 🔁 Build Triggers

### Poll SCM

```groovy
triggers {
  pollSCM('* * * * *')
}
```

### GitHub Webhook (Preferred)

* Faster
* Real-time

---

## 🧩 Pipeline Libraries (Concept)

Reusable pipeline code:

```groovy
@Library('shared-lib') _
```

---

## 🤖 Jenkins Agents Types

| Agent            | Use        |
| ---------------- | ---------- |
| SSH Agent        | Remote VM  |
| Docker Agent     | Containers |
| Kubernetes Agent | Cloud      |

---

## 🚀 Deployment Example (Concept)

```groovy
stage('Deploy') {
  steps {
    sh 'scp app.jar server:/opt/app'
  }
}
```

---

# 💾 Backup & Restore Jenkins

---

## Backup

```bash
docker stop jenkins
docker run --rm -v jenkins_home:/data -v %cd%:/backup busybox tar czf /backup/jenkins-backup.tar.gz /data
```

## Restore

Reverse the process.

---

# ✅ Jenkins Pipeline Best Practices

✔ Always use Jenkinsfile
✔ Use credentials store
✔ Keep pipelines small
✔ Separate build & deploy
✔ Use Docker agents

---

# 🎓 Final Outcome

After this unit, students can:

✅ Setup Jenkins anywhere
✅ Write production-ready pipelines
✅ Integrate Git, Docker & Maven
✅ Understand real-world CI/CD

---
