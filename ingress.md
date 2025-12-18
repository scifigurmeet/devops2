
---

# 🌐 Kubernetes Ingress – Minimal Practical Guide

## 🎯 Objective

Create **one simple Ingress route** that exposes an NGINX application using a hostname:

```
http://web.local
```

---

## 🧠 What is Ingress?

**Ingress** is a Kubernetes object that manages **external HTTP/HTTPS access** to services inside a cluster.

Instead of exposing every service using NodePort or LoadBalancer, Ingress provides:

* A **single entry point**
* **URL / Host-based routing**
* Optional **HTTPS (TLS)**

### 📌 Simple Flow

```
User → Ingress → Service → Pod
```

---

## 🔧 Prerequisites

* Windows 11
* Docker Desktop installed
* Minikube installed
* kubectl installed

Verify:

```bash
minikube version
kubectl version --client
```

---

## 1️⃣ Start Minikube

```bash
minikube start
```

---

## 2️⃣ Enable Ingress Controller (NGINX)

```bash
minikube addons enable ingress
minikube tunnel
```

⚠️ **Keep this terminal open** (tunnel must keep running)

---

## 3️⃣ Create a Sample Application (NGINX)

### Create Deployment

```bash
kubectl create deployment web --image=nginx
```

### Expose as ClusterIP Service

```bash
kubectl expose deployment web --port=80
```

### Verify

```bash
kubectl get pods
kubectl get svc
```

---

## 4️⃣ Create Ingress Resource (Single Route)

📄 Create file: **web-ingress.yaml**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: web.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f web-ingress.yaml
```

Verify:

```bash
kubectl get ingress
```

---

## 5️⃣ Configure Hostname Mapping (Windows)

Edit **hosts file** as Administrator:

```
C:\Windows\System32\drivers\etc\hosts
```

Add:

```
127.0.0.1  web.local
```

Save the file.

---

## 6️⃣ Access Application in Browser 🚀

Open browser and visit:

```
http://web.local
```

✅ You should see **NGINX Welcome Page**

---

## 🧠 What Happened Internally?

```
Browser (web.local)
        ↓
Ingress Controller
        ↓
Ingress Rule (/)
        ↓
Service (web)
        ↓
Pod (nginx)
```

---

## 🧩 Key Components Explained

| Component          | Role                  |
| ------------------ | --------------------- |
| Ingress Controller | Handles routing logic |
| Ingress Resource   | Routing rules         |
| Service            | Load balances pods    |
| Pod                | Runs application      |

---

## ❓ Why Ingress Instead of NodePort?

| Feature       | NodePort | Ingress |
| ------------- | -------- | ------- |
| Single Port   | ❌        | ✅       |
| Multiple Apps | ❌        | ✅       |
| Path Routing  | ❌        | ✅       |
| HTTPS         | ❌        | ✅       |

---

## 📝 One-Line Exam Answer

> **Ingress exposes Kubernetes services externally using HTTP/HTTPS routing rules through a single entry point.**

---

## 🧹 Cleanup (Optional)

```bash
kubectl delete ingress web-ingress
kubectl delete svc web
kubectl delete deployment web
minikube stop
```

---

## ✅ Summary

* Ingress provides **smart routing**
* One hostname → many services
* Cleaner than NodePort
* Industry-standard approach

---


Just tell me 👍
