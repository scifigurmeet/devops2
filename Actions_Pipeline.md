
---

# 🚀 Deploy NGINX on AWS EC2 using GitHub Actions

**(Beginner Friendly | Step-by-Step | Real CI/CD)**

---

## 🧠 What Are We Doing?

Whenever code is pushed to GitHub:

```
Git Push
   ↓
GitHub Actions Pipeline
   ↓
SSH into EC2
   ↓
Build & Run NGINX using Docker
   ↓
Website Live on EC2 Public IP
```

---

## 🎯 Learning Outcomes

Students will learn:

* What **CI/CD** is
* What **GitHub Actions** is
* What **EC2** is
* How **SSH-based deployment** works
* How Docker is used in pipelines
* How real-world deployment happens

---

## 📦 Prerequisites

* GitHub account
* AWS account
* Basic Git knowledge

---

# 🧩 PART 1: Create EC2 Instance (AWS)

---

## 🔹 Step 1: Launch EC2

1. Go to **AWS Console → EC2**
2. Click **Launch Instance**

### Configuration

| Setting       | Value                |
| ------------- | -------------------- |
| Name          | nginx-github-actions |
| AMI           | Ubuntu 22.04         |
| Instance Type | t2.micro (free tier) |
| Key Pair      | Create new key pair  |
| Network       | Default VPC          |
| Storage       | Default (8GB)        |

---

## 🔓 Step 2: Security Group (VERY IMPORTANT)

Allow these inbound rules:

| Type | Port | Source    |
| ---- | ---- | --------- |
| SSH  | 22   | Your IP   |
| HTTP | 80   | 0.0.0.0/0 |

---

## 🔐 Step 3: Save Key Pair

* Download `nginx-key.pem`
* Keep it safe (you’ll need it)

---

# 🧩 PART 2: Prepare EC2 for Deployment

---

## 🔹 Step 4: Connect to EC2

```bash
ssh -i nginx-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

## 🔹 Step 5: Install Docker on EC2

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

➡️ Logout and login again

```bash
exit
ssh -i nginx-key.pem ubuntu@<EC2_PUBLIC_IP>
```

Verify:

```bash
docker --version
```

---

# 🧩 PART 3: Create GitHub Project

---

## 📁 Project Structure

```
nginx-github-actions/
│
├── index.html
├── Dockerfile
└── .github/
    └── workflows/
        └── deploy.yml
```

---

## 🧾 Step 6: `index.html`

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>🚀 NGINX Deployed using GitHub Actions</h1>
    <p>CI/CD is working successfully!</p>
  </body>
</html>
```

---

## 🧾 Step 7: `Dockerfile`

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```

---

# 🧩 PART 4: GitHub Actions Pipeline

---

## 🧾 Step 8: Create Workflow File

Path:

```
.github/workflows/deploy.yml
```

### 📄 `deploy.yml`

```yaml
name: Deploy NGINX to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Deploy to EC2
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.EC2_PUBLIC_IP }}
        username: ubuntu
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          docker stop nginx || true
          docker rm nginx || true
          docker build -t nginx-ci .
          docker run -d -p 80:80 --name nginx nginx-ci
```

---

# 🧩 PART 5: GitHub Secrets Configuration

---

## 🔐 Step 9: Add GitHub Secrets

Go to:

```
GitHub Repo → Settings → Secrets → Actions → New Repository Secret
```

### Add These Secrets

| Secret Name     | Value                      |
| --------------- | -------------------------- |
| `EC2_PUBLIC_IP` | EC2 public IP              |
| `EC2_SSH_KEY`   | Private key (.pem content) |

---

### 🔑 How to Get Private Key Content

```bash
cat nginx-key.pem
```

Copy **everything** including:

```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

---

# 🧩 PART 6: Deploy 🚀

---

## 🔹 Step 10: Push Code

```bash
git init
git add .
git commit -m "Deploy nginx via GitHub Actions"
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>
git push -u origin main
```

---

## 🔹 Step 11: Verify Pipeline

* Go to **GitHub → Actions**
* Workflow should be **green ✅**

---

## 🌐 Step 12: Access Website

Open browser:

```
http://<EC2_PUBLIC_IP>
```

You should see:

> 🚀 NGINX Deployed using GitHub Actions

---

# 🧠 How This Works (Simple Explanation)

1. GitHub detects push to `main`
2. GitHub Actions runner starts
3. Runner SSHs into EC2
4. Builds Docker image on EC2
5. Runs NGINX container
6. Website updates automatically

---

# ❌ Common Errors & Fixes

### ❌ Permission denied (docker)

```bash
sudo usermod -aG docker ubuntu
```

### ❌ Port 80 not opening

* Check EC2 security group

### ❌ SSH connection failed

* Wrong IP
* Wrong key
* Ensure `ubuntu` user

---

# 🎓 Teaching Flow (Recommended)

1. Manual NGINX run on EC2
2. Dockerize NGINX
3. Manual Docker run
4. GitHub Actions deployment
5. Change HTML → auto deploy demo 🎯

---

# 🚀 Next Enhancements (Optional)

* Add **Docker Hub push**
* Add **NGINX SSL**
* Add **Rollback**
* Deploy to **Kubernetes**
* Blue-Green deployment

---


Just say the word 👍
