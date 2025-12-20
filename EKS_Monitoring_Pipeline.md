
---

# 📘 End-to-End Monitoring Pipeline for Kubernetes & EKS

**(Practical Bare Minimum – REAL AWS EKS Cluster)**

---

## 🎯 What You Will Build (End-to-End)

You will create this **complete monitoring pipeline** 👇

![Image](https://d1.awsstatic.com/onedam/marketing-channels/website/aws/en_US/solution-case-studies/approved/images/5b5e4e8a08f513b339e3fb2346471f2d-monitoring-amazon-eks-workloads-with-managed-prometheus-grafana-2982x1824.661869920697df04e7937494555b8d77edb51031.png?utm_source=chatgpt.com)

![Image](https://devopscube.com/content/images/2025/06/loki-component.png?utm_source=chatgpt.com)

![Image](https://platform9.com/media/kubernetes-ci-cd-with-artifactory-helm.png?utm_source=chatgpt.com)

```
EKS Cluster
   ↓
Prometheus (metrics)
   ↓
Grafana (visualization)

Pods
   ↓
Loki (logs)
   ↓
Grafana

EKS / EC2
   ↓
CloudWatch (infra metrics)
```

---

## 🧠 What This Covers (Bare Minimum)

✔ Real **AWS EKS cluster**
✔ Prometheus + Grafana
✔ Application metrics
✔ Logs via Loki
✔ AWS CloudWatch (infra)
✔ No Operators, no CRD theory overload

---

## 💰 Cost & Free Tier Reality (IMPORTANT)

* **EKS control plane** → ~$0.10/hour ⚠️
* Worker nodes → **t3.micro** (Free Tier-ish but still billed)
* **DO THIS PRACTICAL → DELETE CLUSTER SAME DAY**

---

## 🧰 Prerequisites (One Time)

Install locally:

* AWS CLI
* kubectl
* eksctl
* Helm

Verify:

```bash
aws --version
kubectl version --client
eksctl version
helm version
```

---

## ☁️ Step 1: Create a REAL EKS Cluster (Minimal)

We use **eksctl** (simplest way).

```bash
eksctl create cluster --name monitoring-cluster --region ap-south-1 --nodegroup-name ng-1 --node-type t3.micro --nodes 2
```

⏳ Takes ~15 minutes

Verify:

```bash
kubectl get nodes
```

Expected:

```
2 nodes Ready
```

---

## 📊 Step 2: Install Prometheus & Grafana (Helm)

### Add Helm repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Install kube-prometheus-stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

Verify:

```bash
kubectl get pods
```

---

## 📈 Step 3: Access Grafana

Get Grafana password:

```bash
kubectl get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

Port-forward:

```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```

Open browser:

```
http://localhost:3000
```

Login:

* User: `admin`
* Password: (from above)

✅ **Grafana is already connected to Prometheus**

---

## 🧠 What You Are Monitoring Right Now

You already have:

* Cluster CPU
* Node memory
* Pod health
* API server metrics

👉 **No extra config needed**

---

## 🌐 Step 4: Deploy a Sample App (Metrics Target)

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80
```

Verify:

```bash
kubectl get svc nginx
```

---

## 🪵 Step 5: Add Logs Using Loki (Minimal)

### Install Loki

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

```bash
helm install loki grafana/loki-stack --set promtail.enabled=true
```

Verify:

```bash
kubectl get pods | grep loki
```

---

## 📜 Step 6: View Logs in Grafana

In **Grafana UI**:

1. Go to **Explore**
2. Select **Loki**
3. Query:

```
{app="nginx"}
```

🎉 You now see **Kubernetes logs**

---

## ☁️ Step 7: AWS CloudWatch (Infra Monitoring)

EKS automatically sends:

* Node health
* Control plane logs
* EC2 metrics

Go to:
**AWS Console → CloudWatch → Metrics → EKS**

👉 This is **infra-level monitoring**, not app-level

---

## 🔄 End-to-End Data Flow (Final Picture)

```
Pods → Prometheus → Grafana (metrics)
Pods → Loki → Grafana (logs)
EKS → CloudWatch (infra)
```

This is **real-world production architecture**.

---

## 🧠 Key Concepts (Exam + Interview Gold)

* Prometheus = metrics
* Grafana = visualization
* Loki = logs
* CloudWatch = AWS infra
* Helm = package manager
* EKS = managed Kubernetes

---

## 🧹 Step 8: CLEANUP (VERY IMPORTANT)

```bash
eksctl delete cluster --name monitoring-cluster --region ap-south-1
```

💸 **Do NOT skip this**

---

## 🚀 What You Just Achieved

✔ Built a real EKS cluster
✔ Installed monitoring stack
✔ Collected metrics & logs
✔ Used CloudWatch
✔ Understood full pipeline

---

## 📌 What to Teach Next (Optional)

* Alerts in Grafana
* Prometheus rules
* OpenTelemetry + EKS
* Production-grade monitoring
* Cost optimization

---

### ✅ This guide is:

✔ Practical
✔ Beginner-safe
✔ Industry-standard
✔ Classroom-ready
