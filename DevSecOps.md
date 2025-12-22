
---

# 📘 DevSecOps – Beginner to Practical Guide

**(Hands-On | Step-by-Step | Minimal but Complete)**

---

## 🎯 Who This Guide Is For

This guide is for learners who:

* Know **Git & GitHub**
* Have used **CI/CD pipelines** (GitHub Actions, Jenkins basics)
* Know **Docker & containers** at a basic level
* Want to understand **how security fits into DevOps → DevSecOps**

---

## 🧠 What is DevSecOps?

### 🔁 DevOps vs DevSecOps

| DevOps                 | DevSecOps                           |
| ---------------------- | ----------------------------------- |
| Security is at the end | Security is **built-in from start** |
| Manual security checks | **Automated security**              |
| Reactive               | **Proactive**                       |
| Slower fixes           | Early & cheaper fixes               |

👉 **DevSecOps = Development + Security + Operations**

---

## 🔐 Core DevSecOps Idea (Very Important)

> **“Shift Security Left”**

This means:

* Catch security issues **early**
* Automate security checks in CI/CD
* Developers are also responsible for security

---

## 🧱 DevSecOps Pillars

1. **Code Security (SAST)**
2. **Dependency Security (SCA)**
3. **Secrets Management**
4. **Container Security**
5. **Infrastructure Security**
6. **Runtime & Monitoring**

---

# 🧩 DevSecOps Tool Map (Beginner Friendly)

| Stage        | What                   | Tools                         |
| ------------ | ---------------------- | ----------------------------- |
| Code         | Static analysis        | SonarQube                     |
| Dependencies | Vulnerable libraries   | Trivy, OWASP Dependency-Check |
| Secrets      | API keys, passwords    | GitHub Secret Scan            |
| Container    | Image scanning         | Trivy                         |
| IaC          | Terraform/K8s security | Checkov                       |
| CI/CD        | Automation             | GitHub Actions                |
| Runtime      | Logs & alerts          | Prometheus, Grafana           |

---

# 🧪 Practical Setup (Local + GitHub)

---

## ✅ Prerequisites

Install locally:

```bash
git --version
docker --version
```

GitHub account required.

---

# 🧪 LAB 1: Vulnerable Application (Base App)

### Step 1: Create a GitHub Repo

```bash
mkdir devsecops-demo
cd devsecops-demo
git init
```

---

### Step 2: Create a Sample App

**app.py**

```python
import os
print("App Running")
print(os.getenv("API_KEY"))
```

---

### Step 3: Add Vulnerable Dependency

**requirements.txt**

```txt
flask==1.0
requests==2.19.0
```

(These versions contain known vulnerabilities)

---

### Step 4: Commit Code

```bash
git add .
git commit -m "Initial vulnerable app"
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>
git push -u origin main
```

---

# 🔍 LAB 2: Static Application Security Testing (SAST)

## What is SAST?

* Analyzes **source code**
* No execution required
* Finds:

  * Hardcoded secrets
  * Bad coding practices

---

## Tool: SonarQube

We’ll use **SonarCloud** (Cloud version).

### Step 1: Connect GitHub Repo to SonarCloud

* Login to SonarCloud
* Import GitHub repository
* Generate token

---

### Step 2: Add GitHub Secret

In GitHub → Settings → Secrets → Actions

```
SONAR_TOKEN = <token>
```

---

### Step 3: Add GitHub Actions Workflow

**.github/workflows/sonar.yml**

```yaml
name: Sonar Scan

on: [push]

jobs:
  sonar:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Sonar Scan
      uses: SonarSource/sonarcloud-github-action@v2
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

### ✅ Result

* Code smells
* Security issues
* Vulnerability rating

---

# 📦 LAB 3: Dependency Security (SCA)

## What is SCA?

* Scans **third-party libraries**
* Most real attacks come from **dependencies**

---

## Tool: Trivy

### Step 1: Run Locally

```bash
docker run --rm -v $(pwd):/app aquasec/trivy fs /app
```

For Windows:
```bash
docker run --rm -v "%cd%":/app aquasec/trivy fs /app
```

---

### What You’ll See

* CVE IDs
* Severity (LOW → CRITICAL)
* Affected library versions

---

### Step 2: Fix Dependencies

Update **requirements.txt**

```txt
flask>=2.0
requests>=2.31.0
```

---

# 🐳 LAB 4: Container Security

## Step 1: Create Dockerfile

```dockerfile
FROM python:3.8
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

---

## Step 2: Build Image

```bash
docker build -t devsecops-demo .
```

---

## Step 3: Scan Image

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image devsecops-demo
```

---

### 🧠 What This Detects

* OS vulnerabilities
* Python package vulnerabilities

---

# 🔑 LAB 5: Secrets Management (Very Important)

## ❌ BAD PRACTICE

```python
API_KEY="123456"
```

---

## ✅ Correct Way

### Step 1: Remove Secrets from Code

```python
os.getenv("API_KEY")
```

---

### Step 2: Add GitHub Secret

```
API_KEY = supersecretkey
```

---

### Step 3: Use in Workflow

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

---

# 🧱 LAB 6: Infrastructure as Code Security (IaC)

## What is IaC Security?

* Scan Terraform / Kubernetes YAML
* Prevent:

  * Open security groups
  * Privileged containers

---

## Tool: Checkov

### Step 1: Sample Terraform

```hcl
resource "aws_security_group" "bad" {
  ingress {
    from_port = 0
    to_port   = 65535
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### Step 2: Scan

```bash
docker run --rm -v $(pwd):/tf bridgecrew/checkov -d /tf
```

---

# 🚀 LAB 7: DevSecOps CI/CD Pipeline (Full)

**.github/workflows/devsecops.yml**

```yaml
name: DevSecOps Pipeline

on: [push]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Trivy FS Scan
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: fs
        severity: HIGH,CRITICAL

    - name: Build Image
      run: docker build -t app .

    - name: Trivy Image Scan
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: app
```

---

# 📊 LAB 8: Monitoring & Runtime Security (Intro)

## What Happens After Deployment?

* Logs
* Alerts
* Attack detection

---

### Tools Overview

* Prometheus → Metrics
* Grafana → Visualization
* Falco → Runtime security

---

# 📌 DevSecOps Maturity Levels

| Level   | Description        |
| ------- | ------------------ |
| Level 1 | Manual scans       |
| Level 2 | CI security checks |
| Level 3 | Policy enforcement |
| Level 4 | Runtime protection |

---

# 📚 Interview-Ready DevSecOps Concepts

* Shift Left Security
* SAST vs DAST vs SCA
* Secrets management
* Zero Trust
* Least Privilege
* SBOM

---

# 🧠 Common Beginner Mistakes

❌ Scanning only once
❌ Hardcoding secrets
❌ Ignoring dependency risks
❌ No security gates in CI

---

# ✅ Final Takeaway

> **DevSecOps is not extra work — it is smarter DevOps**

Security must be:

* Automated
* Continuous
* Developer-friendly

---
