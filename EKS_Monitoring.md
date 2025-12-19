
---

# 📊 Integrating Prometheus and Grafana in Amazon EKS

**(Beginner Friendly | Step-by-Step | Browser Accessible)**

---

## 🎯 Objective

By the end of this lab, you will be able to:

* Deploy **Prometheus** on EKS
* Deploy **Grafana** on EKS
* Connect Grafana to Prometheus
* Open Grafana in a browser
* View **basic Kubernetes cluster metrics**

---

## 🧠 High-Level Architecture (Simple)

```
EKS Cluster
 ├── Prometheus (collects metrics)
 └── Grafana (visualizes metrics)
```

Grafana **reads data from Prometheus** and shows graphs.

---

## 🧰 Prerequisites

### 1️⃣ Tools Required (Install Before Starting)

* AWS Account
* AWS CLI
* eksctl
* kubectl
* Helm

### Verify installations

```bash
aws --version
kubectl version --client
eksctl version
helm version
```

---

## ☁️ Step 1: Create an EKS Cluster (Minimal)

> ⚠️ This creates AWS resources → costs may apply

```bash
eksctl create cluster --name monitoring-cluster --region ap-south-1 --nodegroup-name workers --node-type t3.medium --nodes 2
```

⏳ Wait 10–15 minutes.

### Verify cluster

```bash
kubectl get nodes
```

You should see **2 worker nodes** in `Ready` state.

---

## 📦 Step 2: Add Helm Repositories

Helm makes installation **very easy**.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

---

## 📈 Step 3: Install Prometheus (Basic)

We’ll install **Prometheus + Kubernetes metrics**.

```bash
helm install prometheus prometheus-community/prometheus
```

### Verify Prometheus Pods

```bash
kubectl get pods
```

Look for pods like:

* prometheus-server
* prometheus-kube-state-metrics
* prometheus-node-exporter

---

## 📊 Step 4: Install Grafana (Basic)

```bash
helm install grafana grafana/grafana
```

### Check Grafana Pod

```bash
kubectl get pods
```

You should see:

```
grafana-xxxxx   Running
```

---

## 🔐 Step 5: Get Grafana Admin Password

Grafana stores password in a Kubernetes Secret.

```bash
kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

📌 **Save this password**

---

## 🌐 Step 6: Expose Grafana to Browser (Easy Way)

We’ll use **port-forwarding** (safe and simple).

```bash
kubectl port-forward svc/grafana 3000:80
```

### Open Browser

```
http://localhost:3000
```

### Login

* **Username:** `admin`
* **Password:** (from previous step)

---

## 🔗 Step 7: Add Prometheus as Data Source in Grafana

### In Grafana UI:

1. Click **⚙️ Settings**
2. Click **Data Sources**
3. Click **Add data source**
4. Choose **Prometheus**
5. Set URL:

```
http://prometheus-server
```

6. Click **Save & Test**

✅ You should see **“Data source is working”**

---

## 📉 Step 8: Import a Ready-Made Dashboard (Fast Win 🎉)

### Import Kubernetes Dashboard

1. Click **+ → Import**
2. Enter Dashboard ID:

```
6417
```

3. Select **Prometheus** as data source
4. Click **Import**

🎉 You will now see:

* CPU usage
* Memory usage
* Node metrics
* Pod metrics

---

## 🔍 Step 9: Verify Metrics Are Working

Go to **Explore → Prometheus** and try:

```promql
up
```

You should see **multiple targets = UP**

---

## 🧪 Optional: Deploy Sample App (To Observe Metrics)

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
```

Now watch:

* Pod count increase
* CPU usage change
* Memory graphs update

---

## 🧹 Cleanup (Important!)

To avoid AWS charges:

```bash
eksctl delete cluster --name monitoring-cluster --region ap-south-1
```

---

## ✅ Summary

| Component  | Purpose             |
| ---------- | ------------------- |
| Prometheus | Collects metrics    |
| Grafana    | Visualizes metrics  |
| Helm       | Simplifies installs |
| EKS        | Managed Kubernetes  |

---

## 📌 What You Can Teach With This

* Real-world monitoring stack
* Production-grade tooling
* Cloud + Kubernetes observability
* Visual metrics understanding

---

Just tell me 👍
