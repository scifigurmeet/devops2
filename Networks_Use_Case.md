
---

# 🚀 **Docker Networks – Minimal Real-World Use Case (Beginner’s Reference Guide)**

A simple guide to understand Docker networks using a real-world example:
👉 **A Web App (Nginx) talking to an API (Python Flask)** inside a custom Docker network.

---

## 📌 **What Are Docker Networks?**

Docker networks allow containers to talk to each other **securely and predictably**.
When containers share the same network, they can communicate using **container names as hostnames**.

Example:
`web` → can reach → `api`
because both are in the same network.

---

# 🌐 Create a Real-World Multi-Container Setup

We will create:

* **api** → A small Python Flask API returning "Hello from API"
* **web** → Nginx serving a webpage that calls the API
* **my-network** → A custom Docker bridge network to connect them

---

# 🛠️ **Step 1: Create Network**

```bash
docker network create my-network
```

---

# 🧱 **Step 2: Create API Container (Flask)**

Create a file named **app.py**:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from API!"

app.run(host='0.0.0.0', port=5000)
```

Create a **Dockerfile**:

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
CMD ["python", "app.py"]
```

Build the image:

```bash
docker build -t simple-api .
```

Run the API container **on the custom network**:

```bash
docker run -d --name api --network my-network -p 5000:5000 simple-api
```

---

# 🌍 **Step 3: Create Web Container (Nginx)**

We will create a simple static HTML page that calls the API.

Create **index.html**:

```html
<h1>Web App</h1>
<p id="msg">Loading...</p>

<script>
  fetch("http://api:5000")
    .then(res => res.text())
    .then(data => document.getElementById("msg").innerText = data)
    .catch(() => document.getElementById("msg").innerText = "API not reachable");
</script>
```

Create **Dockerfile**:

```Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

Build:

```bash
docker build -t simple-web .
```

Run:

```bash
docker run -d --name web --network my-network -p 8080:80 simple-web
```

---

# ✅ **Step 4: Test the Real-World Use Case**

### Visit:

👉 **[http://localhost:8080](http://localhost:8080)**

You will see:

```
Web App
Hello from API!
```

This proves:

* **web** container can reach **api** container
* Communication happens via Docker **network name → hostname resolution**

---

# 📘 **Network Commands — Quick Reference**

### 🔍 List networks

```bash
docker network ls
```

### 📑 Inspect a network

```bash
docker network inspect my-network
```

### ➕ Connect a container to a network

```bash
docker network connect my-network web
```

### ➖ Disconnect a container

```bash
docker network disconnect my-network web
```

### 🗑 Delete a network

```bash
docker network rm my-network
```

### 🧹 Remove all unused networks

```bash
docker network prune
```

---

# 🎉 **You Just Built a Two-Container Microservice System!**

This real-world example demonstrates:

✔ Web-to-API communication
✔ Docker DNS hostname resolution
✔ How networks isolate and connect services
✔ A practical microservices foundation

Just say **“compose version”** or **“add database example”**!
