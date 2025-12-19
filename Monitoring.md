
---

# 📘 Unit IV – Monitoring in Kubernetes using Prometheus & Grafana

**(Beginner Friendly | Windows + Minikube Only)**

---

## 🎯 What Students Will Learn

By the end of this unit, students will be able to:

* Understand **why monitoring is needed**
* Understand **what Prometheus and Grafana are**
* Install **Prometheus & Grafana on Minikube**
* Collect **Kubernetes metrics**
* Connect **Grafana with Prometheus**
* Build **basic dashboards** for:

  * Cluster monitoring
  * Application monitoring

---

## 🧠 First: What is Monitoring? (Very Important)

### ❓ What is Monitoring?

Monitoring means:

> **Continuously checking the health and performance of systems**

In Kubernetes, we want to know:

* Are pods running?
* Is CPU or memory high?
* Is the application slow?
* Did something crash?

---

## 🏭 Real-Life Example (Simple)

Think of Kubernetes like a **factory**:

| Kubernetes | Real Life    |
| ---------- | ------------ |
| Pod        | Machine      |
| Node       | Floor        |
| Cluster    | Factory      |
| CPU/Memory | Electricity  |
| Monitoring | Control Room |

📊 **Prometheus = Sensor System**
📈 **Grafana = Big Display Screen**

---

## 🔍 Monitoring in Kubernetes Environment

In Kubernetes, monitoring tracks:

* Pod CPU & memory
* Node health
* Application response
* Restart counts
* Resource usage

---

## 🧱 Monitoring Stack (We Will Use)

```
Kubernetes Cluster
       ↓
   Prometheus
       ↓
     Grafana
```

---

## 🚀 What is Prometheus?

### 📌 Definition

**Prometheus** is an **open-source monitoring and metrics collection system**.

### 🧠 What Prometheus Does

* Collects metrics (numbers like CPU, memory)
* Stores them as **time-series data**
* Uses **pull-based model** (it pulls metrics)

---

## 🏗️ Prometheus Architecture (Simple)

```
Targets (Pods, Nodes)
        ↑
   Service Discovery
        ↑
    Prometheus Server
        ↓
     Time Series DB
```

### 🔑 Key Components

| Component         | Purpose                                 |
| ----------------- | --------------------------------------- |
| Prometheus Server | Collects & stores metrics               |
| Service Discovery | Finds Kubernetes services automatically |
| Metrics Endpoint  | `/metrics` URL                          |
| TSDB              | Time-series database                    |

---

## ⭐ Features of Prometheus

* Pull-based metrics
* Kubernetes native
* Auto service discovery
* Powerful query language (PromQL)
* Free & open-source

---

## ⚙️ Installing Prometheus on Minikube (Easy Way)

We will use **Helm** (recommended for beginners).

### ✅ Step 1: Verify Minikube

```cmd
minikube status
```

If not running:

```cmd
minikube start
```

---

### ✅ Step 2: Enable Helm (Already available in Minikube)

Check:

```cmd
helm version
```

---

### ✅ Step 3: Add Prometheus Helm Repo

```cmd
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

### ✅ Step 4: Install Prometheus

```cmd
helm install prometheus prometheus-community/prometheus
```

---

### ✅ Step 5: Verify Prometheus Pods

```cmd
kubectl get pods
```

You should see pods like:

```
prometheus-server
prometheus-kube-state-metrics
prometheus-node-exporter
```

---

## 🌐 Access Prometheus UI

### Use Port Forwarding (Windows Friendly)

```cmd
kubectl port-forward svc/prometheus-server 9090:80
```

Open browser:

```
http://localhost:9090
```

🎉 Prometheus is running!

---

## 📊 What Metrics Does Prometheus Collect?

Examples:

* `node_cpu_seconds_total`
* `container_memory_usage_bytes`
* `kube_pod_status_phase`

👉 In Prometheus UI:

* Go to **Graph**
* Type:

```
up
```

If result = 1 → Target is healthy

---

## 🔍 What is Service Discovery?

### ❓ Problem Without Discovery

Manually adding pod IPs ❌

### ✅ Solution

Prometheus **automatically finds**:

* Pods
* Nodes
* Services

Using Kubernetes API 🎯

---

## 🎨 What is Grafana?

### 📌 Definition

**Grafana** is an **open-source visualization tool**.

Prometheus gives **numbers**
Grafana turns them into **charts & dashboards**

---

## ⭐ Grafana Features

* Beautiful dashboards
* Supports Prometheus
* Alerts
* Real-time graphs
* Easy UI

---

## ⚙️ Install Grafana on Minikube

### Step 1: Add Grafana Repo

```cmd
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

---

### Step 2: Install Grafana

```cmd
helm install grafana grafana/grafana
```

---

### Step 3: Get Grafana Admin Password

```cmd
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

📌 **Username:** `admin`
📌 **Password:** (output of above command)

---

### Step 4: Access Grafana UI

```cmd
kubectl port-forward svc/grafana 3000:80
```

Open browser:

```
http://localhost:3000
```

---

## 🔗 Connect Grafana with Prometheus

### Step 1: Login to Grafana

### Step 2: Add Data Source

* Go to **Settings → Data Sources**
* Click **Add data source**
* Select **Prometheus**

### Step 3: Set URL

```
http://prometheus-server
```

Click **Save & Test** ✅

---

## 📈 Build First Dashboard (Cluster Monitoring)

### Step 1: Create Dashboard

* Click **+ → Dashboard**
* Add new panel

---

### Step 2: Example CPU Query

```promql
sum(rate(container_cpu_usage_seconds_total[1m]))
```

📊 Visualization: Graph
🕒 Time range: Last 5 minutes

---

### Step 3: Memory Usage Query

```promql
sum(container_memory_usage_bytes) / (1024 * 1024)
```

Displays memory in **MB**

---

## 🌍 Application Monitoring Example (Nginx)

### Step 1: Deploy Nginx

```cmd
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

---

### Step 2: Verify Pod

```cmd
kubectl get pods
```

---

### Step 3: Prometheus Automatically Discovers It 🎯

Grafana can now show:

* Pod CPU
* Pod memory
* Restart count

Example Query:

```promql
kube_pod_container_status_restarts_total
```

---

## 📊 Common Useful Metrics for Teaching

| Metric                              | Purpose        |
| ----------------------------------- | -------------- |
| `up`                                | Service health |
| `node_memory_MemAvailable_bytes`    | Node memory    |
| `container_cpu_usage_seconds_total` | CPU usage      |
| `kube_pod_status_phase`             | Pod state      |

---

## 🧹 Cleanup (Optional)

```cmd
helm uninstall prometheus
helm uninstall grafana
kubectl delete deployment nginx
kubectl delete service nginx
```

---

## 🧠 Final Mental Model (Explain to Students)

* **Kubernetes** runs apps
* **Prometheus** watches Kubernetes
* **Grafana** shows what Prometheus sees

📌 If something crashes → Prometheus knows
📌 If CPU is high → Grafana shows graph

---

## ✅ Unit IV Completed ✔️

Students now understand:

* Monitoring basics
* Prometheus architecture
* Metrics collection
* Grafana dashboards
* Real Kubernetes monitoring

---
