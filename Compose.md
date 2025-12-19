
---

# 🐳 Docker Compose – Complete Beginner Study Guide

**(One Guide is Enough | Minimal Examples | Practical First)**

---

## 🎯 Who is this for?

This guide is for students who:

* Know **basic Docker** (image, container)
* Are **new to Docker Compose**
* Use **Windows 11 + Docker Desktop**

No prior YAML or DevOps knowledge assumed.

---

## 📌 What Students Will Learn

By the end of this guide, students will:

* Understand **what problem Docker Compose solves**
* Understand **docker-compose.yml**
* Run **multi-container applications**
* Use **services, networks, volumes**
* Scale containers
* Use **environment variables**
* Understand **Compose lifecycle commands**
* Run a **real-world mini project**

---

## 🧠 First: Why Docker Compose?

### ❌ Problem Without Docker Compose

Imagine an app needs:

* 1️⃣ Nginx
* 2️⃣ Backend API
* 3️⃣ Database (MySQL)

Without Compose:

```bash
docker run ...
docker run ...
docker run ...
```

❌ Hard to manage
❌ Hard to share setup
❌ No single source of truth

---

### ✅ Solution: Docker Compose

**Docker Compose = Define everything in ONE file**

📄 `docker-compose.yml`

Then run:

```bash
docker compose up
```

💡 Entire application starts together.

---

## 🏗️ Docker Compose Architecture (Simple)

* **Service** → A container definition
* **Network** → How services talk
* **Volume** → Persistent storage

Think like:

```
Application
 ├── Service 1 (Web)
 ├── Service 2 (API)
 └── Service 3 (Database)
```

---

## 📂 Project Structure (Always Follow This)

```
docker-compose-demo/
│
├── docker-compose.yml
└── index.html   (optional)
```

---

## 🧾 docker-compose.yml Basics

### Minimal File Structure

```yaml
version: "3.9"

services:
  service-name:
    image: image-name
```

---

## 🚀 Example 1: Single Container using Docker Compose

### Step 1: Create project folder

```bash
mkdir compose-basic
cd compose-basic
```

---

### Step 2: Create `docker-compose.yml`

```yaml
version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

---

### Step 3: Start container

```bash
docker compose up
```

Open browser:

```
http://localhost:8080
```

🎉 Nginx running via Docker Compose

---

### Step 4: Stop container

```bash
docker compose down
```

---

## 🔄 docker compose up vs down

| Command                | Purpose             |
| ---------------------- | ------------------- |
| `docker compose up`    | Start services      |
| `docker compose up -d` | Start in background |
| `docker compose down`  | Stop & remove       |

---

## 🧱 Example 2: Custom HTML Website

### Step 1: Create `index.html`

```html
<h1>Hello from Docker Compose</h1>
<p>This is a custom page</p>
```

---

### Step 2: Update `docker-compose.yml`

```yaml
version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - .:/usr/share/nginx/html
```

---

### Step 3: Run

```bash
docker compose up
```

Refresh browser → see your HTML

---

## 💾 What is a Volume?

Volume = **Folder sharing between host & container**

```
Host Folder  --->  Container Folder
```

Used for:

* Data persistence
* Live code changes

---

## 🌐 Example 3: Multi-Container App (Web + Database)

### Scenario

* Web: Nginx
* DB: MySQL

---

### docker-compose.yml

```yaml
version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "8080:80"

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: appdb
```

---

### Run

```bash
docker compose up
```

💡 Both containers run together

---

## 🔑 Environment Variables Explained

```yaml
environment:
  KEY: value
```

Used for:

* Passwords
* Config values
* App settings

---

## 🌍 How Containers Talk to Each Other

### Important Rule

> **Service name = hostname**

In above example:

* Web can access DB using `db`
* No IP required

---

## 🌐 Docker Compose Networking (Automatic)

* Docker Compose creates **one private network**
* All services join it
* DNS works automatically

No extra configuration needed.

---

## 📦 Example 4: Volumes for Database Persistence

```yaml
version: "3.9"

services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

💡 Data survives container restart

---

## 📈 Example 5: Scaling Services

```yaml
services:
  web:
    image: nginx
```

Run:

```bash
docker compose up --scale web=3
```

Check:

```bash
docker ps
```

🎯 3 containers of same service

---

## 🧹 Cleanup Commands (Very Important)

```bash
docker compose down
docker compose down -v
docker system prune
```

---

## 🔍 Useful Inspection Commands

```bash
docker compose ps
docker compose logs
docker compose logs web
docker compose exec web sh
```

---

## 🧪 Mini Project: Web + DB App

### Goal

Run:

* Nginx frontend
* MySQL backend
* Persistent data

---

### Final docker-compose.yml

```yaml
version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - .:/usr/share/nginx/html
    depends_on:
      - db

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: school
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

---

## ⛓️ What is `depends_on`?

```yaml
depends_on:
  - db
```

✔ Ensures DB starts before Web
❌ Does NOT check readiness

---

## 📌 Best Practices (Beginner Friendly)

* Use **clear service names**
* Use **ports only when needed**
* Use **volumes for data**
* Never hardcode secrets in real projects
* One project = one `docker-compose.yml`

---

## 🧠 Docker Compose vs Dockerfile

| Dockerfile     | Docker Compose    |
| -------------- | ----------------- |
| Builds image   | Runs containers   |
| Single service | Multi-service     |
| Image recipe   | App orchestration |

---

## 📚 Mental Model (Very Important)

> Docker Compose is like **kubernetes for beginners**

* YAML file
* Declarative
* One command to run everything

---

## ✅ Final Commands Summary

```bash
docker compose up
docker compose up -d
docker compose down
docker compose ps
docker compose logs
```

---

## 🎓 What Students Should Practice Next

* Add backend service (Node/Python)
* Add `.env` file
* Add restart policies
* Try Redis or MongoDB

---

## 🏁 Conclusion

If you understand this guide:

✅ You understand **Docker Compose**
✅ You can run **real applications**
✅ You are ready for **Kubernetes & DevOps**

---
