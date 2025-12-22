
---

# 📘 UNIT 3 – Microservices with Docker Compose

## 🐳 WordPress + MySQL Deployment (Step-by-Step Practical Guide)

---

## 🎯 Learning Objectives

By the end of this unit, you will be able to:

* Understand **Microservices Architecture**
* Compare **Monolithic vs Microservices**
* Understand **Docker Compose**
* Write a **docker-compose.yml** file
* Deploy a **multi-container application**
* Run **WordPress with MySQL** using Docker Compose
* Understand **volumes, networks, environment variables & dependencies**

---

## 🧠 Part 1: Microservices Architecture

### ❓ What is Microservices Architecture?

Microservices architecture is a design approach where:

* An application is broken into **small independent services**
* Each service performs **one specific function**
* Services communicate via **APIs**

Example:

* Frontend service
* Backend service
* Database service

Each runs **independently** and can be scaled separately.

---

### 🏢 Monolithic vs Microservices

| Feature        | Monolithic         | Microservices           |
| -------------- | ------------------ | ----------------------- |
| Structure      | Single big app     | Multiple small services |
| Scaling        | Entire app         | Individual services     |
| Failure Impact | Whole app may fail | Only one service        |
| Deployment     | Slow               | Fast & independent      |
| Technology     | Single stack       | Multiple stacks         |

---

### ✅ Why Microservices? (Advantages)

✔ **Scalability** – Scale only what is needed
✔ **Isolation** – One service crash doesn’t kill all
✔ **Agility** – Faster development & deployment
✔ **Technology Freedom** – Different tech per service
✔ **API Gateway Ready** – Central entry point for services

---

## 🧠 Part 2: What is Docker Compose?

Docker Compose is a tool to:

* Define **multiple containers**
* Configure them using **one YAML file**
* Start everything using **one command**

📄 File name used:

```
docker-compose.yml
```

---

## 🧩 Docker Compose YAML Structure

```yaml
version: "3.9"

services:
  service1:
    image: example
    ports:
      - "8080:80"

volumes:
  myvolume:

networks:
  mynetwork:
```

---

## 🧠 Key Sections Explained

| Section     | Purpose                     |
| ----------- | --------------------------- |
| version     | Compose file version        |
| services    | Containers definition       |
| image       | Docker image to use         |
| build       | Build image from Dockerfile |
| ports       | Port mapping                |
| environment | Environment variables       |
| volumes     | Persistent storage          |
| networks    | Internal communication      |
| depends_on  | Service startup order       |

---

## 🏗️ Architecture: WordPress + MySQL

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AMZOU5mWINKry3XXxAc34GQ.png?utm_source=chatgpt.com)

![Image](https://www.kajabity.com/wordpress/wp-content/uploads/2021/08/wordpress-web-server-mysql-database.png?utm_source=chatgpt.com)

![Image](https://miro.medium.com/1%2A0QAJMIkvfcj6GuYF8Xxsbw.png?utm_source=chatgpt.com)

### Components:

* **WordPress** → Frontend + PHP
* **MySQL** → Database
* Both run in **separate containers**
* Connected via **Docker network**

---

## 🧪 Part 3: Practical – WordPress + MySQL using Docker Compose

---

## ✅ Prerequisites

Make sure you have installed:

```bash
docker --version
docker compose version
```

> If Docker Desktop is installed → Docker Compose is already included

---

## 📁 Step 1: Create Project Directory

```bash
mkdir wordpress-compose
cd wordpress-compose
```

---

## 📄 Step 2: Create `docker-compose.yml`

Create a file named **docker-compose.yml**

```yaml
version: "3.9"

services:
  mysql:
    image: mysql:8.0
    container_name: wordpress-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - wp-network

  wordpress:
    image: wordpress:latest
    container_name: wordpress-app
    restart: always
    ports:
      - "8080:80"
    # environment:
    #   WORDPRESS_DB_HOST: mysql
    #   WORDPRESS_DB_USER: wpuser
    #   WORDPRESS_DB_PASSWORD: wppass
    #   WORDPRESS_DB_NAME: wordpress
    depends_on:
      - mysql
    networks:
      - wp-network

volumes:
  mysql_data:

networks:
  wp-network:
```

---

## 🧠 Explanation of Key Fields

### 🔹 version

```yaml
version: "3.9"
```

Defines Docker Compose syntax version.

---

### 🔹 services

Defines each container.

```yaml
services:
  mysql:
  wordpress:
```

Each service = **one container**

---

### 🔹 image vs build

```yaml
image: mysql:8.0
```

✔ Pulls image from Docker Hub
❌ No Dockerfile needed

> `build:` is used when **you create your own Dockerfile**

---

### 🔹 environment variables

Used to configure applications **without hardcoding values**

```yaml
environment:
  MYSQL_DATABASE: wordpress
```

---

### 🔹 volumes (Persistence)

```yaml
volumes:
  - mysql_data:/var/lib/mysql
```

✔ Keeps database data even after container restart
✔ Prevents data loss

---

### 🔹 networks

```yaml
networks:
  - wp-network
```

✔ Enables containers to talk via **service name**
✔ WordPress connects to MySQL using hostname `mysql`

---

### 🔹 depends_on

```yaml
depends_on:
  - mysql
```

✔ Starts MySQL **before** WordPress
❗ Does NOT wait for DB to be fully ready (important concept)

---

## ▶️ Step 3: Start the Application

```bash
docker compose up -d
```

---

## 🔍 Step 4: Verify Containers

```bash
docker ps
```

Expected output:

```
wordpress-app
wordpress-mysql
```

---

## 🌐 Step 5: Access WordPress

Open browser:

```
http://localhost:8080
```

You will see **WordPress installation screen** 🎉

---

## 🧪 Step 6: Stop & Cleanup

```bash
docker compose down
```

Remove volumes (⚠ deletes DB data):

```bash
docker compose down -v
```

---

## 🧠 Microservices Concepts Applied Here

| Concept       | How It’s Used                  |
| ------------- | ------------------------------ |
| Microservices | WordPress & MySQL separate     |
| Isolation     | DB failure won’t kill frontend |
| Scalability   | Can scale WordPress container  |
| Networking    | Internal Docker network        |
| Configuration | Environment variables          |
| Persistence   | Volumes                        |

---

## 🧪 Bonus Commands (Very Important)

### View logs

```bash
docker compose logs wordpress
docker compose logs mysql
```

### Restart a service

```bash
docker compose restart wordpress
```

### Scale WordPress

```bash
docker compose up -d --scale wordpress=2
```

---

## 📌 Common Beginner Mistakes

❌ Using `localhost` instead of service name
❌ Forgetting volumes → DB data lost
❌ Wrong DB credentials
❌ Not exposing ports

---

## 🧾 Summary

✔ Learned **Microservices architecture**
✔ Understood **Docker Compose YAML**
✔ Deployed **multi-container app**
✔ Implemented **WordPress + MySQL**
✔ Used **volumes, networks & dependencies**

---

## 📚 Next Use Cases to Practice

* Node.js + MongoDB
* Spring Boot + PostgreSQL
* Frontend + Backend + DB
* API Gateway + Services

---
