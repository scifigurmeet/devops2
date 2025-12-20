
---

# 🚀 Unit VI – Automation & Future Trends: Practical Guide

**For Absolute Beginners | Hands-On Practice (Windows + Minikube)**

---

## 📋 Prerequisites Check

Before starting, verify your setup:

```bat
:: Check Minikube is running
minikube status

:: Check Prometheus pods
kubectl get pods -n default | findstr prometheus

:: Check Grafana pods
kubectl get pods -n default | findstr grafana
```

✅ Expected: All pods should show `Running`

---

## 🎯 What We'll Practice Today

1. ✅ Backup & Restore Grafana Dashboards
2. ✅ Configure Prometheus Data Retention
3. ✅ Basic Security Concepts
4. ✅ Install & Explore Grafana Loki
5. ✅ Troubleshoot Common Issues
6. ✅ Understand OpenTelemetry (Future Trend)

---

## 📦 Part 1: Understanding Your Current Setup with Helm

### 🧠 What is Helm? (Quick Recap)

Helm is like an **App Store for Kubernetes**:

* Installs many Kubernetes resources together
* Uses templates and values
* You already used it for Prometheus & Grafana

---

### 🔍 See What Helm Installed

```bat
:: List all Helm releases
helm list -A

:: See values used for Prometheus
helm get values prometheus -n default

:: See all Kubernetes resources created
helm get manifest prometheus -n default
```

---

## 💾 Part 2: Backup & Restore Grafana Dashboards

### 📤 Method 1: Manual Backup (UI – Recommended for Beginners)

```bat
:: Get Grafana URL
minikube service grafana --url

:: OR port-forward
kubectl port-forward svc/grafana 3000:80 -n default
```

Open browser:
👉 [http://localhost:3000](http://localhost:3000)
(Default login: `admin / admin`)

**Export Dashboard**

1. Open dashboard
2. ⚙️ Settings → JSON Model
3. Copy JSON
4. Save as `dashboard-backup.json`

---

### 📥 Restore Dashboard

1. Grafana → `+` → Import
2. Paste JSON / upload file
3. Click **Import**

---

### 📤 Method 2: CLI-Based Backup (Concept)

```bat
:: Get Grafana admin password
kubectl get secret grafana -n default -o jsonpath="{.data.admin-password}" | base64 --decode
```

🧠 For beginners: **UI backup is enough**
🧠 In industry: dashboards are stored in **Git (Dashboard as Code)**

---

## 🗂️ Part 3: Managing Prometheus Data Retention

### 🧠 What is Retention?

Retention = **How long Prometheus keeps metrics**

| Retention | Meaning            |
| --------- | ------------------ |
| 7d        | Last 7 days only   |
| 15d       | Default            |
| 30d       | Long-term analysis |

---

### 🔍 Check Current Retention (Correct Way)

Retention is **NOT in ConfigMap**
It is in **Prometheus container startup arguments**

```bat
kubectl describe pod -n default | findstr retention
```

OR more precise:

```bat
kubectl get deployment prometheus-server -n default -o yaml | findstr retention
```

If nothing appears → default is **15 days**

---

### ⚙️ Update Retention Using Helm (Correct Way)

```bat
:: Set retention to 7 days
helm upgrade prometheus prometheus-community/prometheus --set server.retention=7d -n default

:: OR set to 30 days
helm upgrade prometheus prometheus-community/prometheus --set server.retention=30d -n default
```

---

### ✅ Verify Change

```bat
kubectl get deployment prometheus-server -n default -o yaml | findstr retention
```

You should see:

```
--storage.tsdb.retention.time=7d
```

---

## 🔐 Part 4: Securing Monitoring Tools

### 🔑 Change Grafana Admin Password

```bat
:: Find Grafana pod
kubectl get pods -n default | findstr grafana
```

Copy pod name, then:

```bat
kubectl exec -it grafana-XXXXX -n default -- /bin/sh
```

Inside pod:

```bash
grafana-cli admin reset-admin-password NewSecurePassword123
exit
```

---

### 🛂 Grafana User Roles

| Role   | Permissions     |
| ------ | --------------- |
| Viewer | Read-only       |
| Editor | Edit dashboards |
| Admin  | Full access     |

(UI-based – no CLI required)

---

## 📜 Part 5: Grafana Loki (Logs)

### 🧠 What is Loki?

| Tool       | Stores  |
| ---------- | ------- |
| Prometheus | Metrics |
| Loki       | Logs    |

---

### 🚀 Install Loki (Windows Friendly)

```bat
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

```bat
helm install loki grafana/loki-stack --set loki.persistence.enabled=false --set promtail.enabled=true -n default
```

---

### ✅ Verify Loki

```bat
kubectl get pods -n default | findstr loki
kubectl get pods -n default | findstr promtail
```

---

### 🔌 Connect Loki to Grafana

```bat
kubectl port-forward svc/grafana 3000:80 -n default
```

Grafana → Data Sources → Add Loki
URL:

```
http://loki:3100
```

---

### 🔍 Loki Queries (Grafana → Explore)

```logql
{namespace="default"}
{namespace="default"} |= "error"
{pod="log-generator"}
```

---

### 🧪 Generate Logs

```bat
kubectl run log-generator --image=busybox --restart=Never -- /bin/sh -c "while true; do echo Log from pod; sleep 5; done"
```

---

## 🔭 Part 6: OpenTelemetry (Conceptual)

### 🧠 Why OpenTelemetry?

Before:

```
Metrics → Prometheus
Logs → Loki
Traces → Jaeger
```

After:

```
App → OpenTelemetry → Any backend
```

**One standard. Future-proof.**

---

### (Optional) Install Collector

```bat
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

```bat
helm install otel-collector open-telemetry/opentelemetry-collector --set mode=deployment -n default
```

---

## 🛠️ Part 7: Troubleshooting (Windows Commands)

### ❌ No Data in Grafana

```bat
kubectl get pods -n default | findstr prometheus
kubectl port-forward svc/prometheus-server 9090:80 -n default
```

Browser:

```
http://localhost:9090/targets
```

---

### ❌ Prometheus Pod Crashing

```bat
kubectl get pods -n default | findstr prometheus
kubectl logs prometheus-server-XXXXX -n default
```

Increase memory:

```bat
helm upgrade prometheus prometheus-community/prometheus --set server.resources.limits.memory=2Gi -n default
```

---

### ❌ Loki Not Showing Logs

```bat
kubectl get pods -n default | findstr promtail
kubectl logs loki-promtail-XXXXX -n default
```

---

## 🌐 Part 8: AWS CloudWatch (Concept)

| CloudWatch    | Prometheus  |
| ------------- | ----------- |
| AWS-only      | Anywhere    |
| Paid          | Free        |
| Infra focused | App focused |

**In real world: use both together**

---

## 🔄 Part 9: End-to-End Monitoring Pipeline

```
Application
   ↓
Metrics → Prometheus → Grafana
Logs → Promtail → Loki → Grafana
Alerts → Alertmanager
```

---

## 📝 Part 10: Final Practice

```bat
kubectl get pods -n default
kubectl port-forward svc/prometheus-server 9090:80 -n default
kubectl port-forward svc/grafana 3000:80 -n default
kubectl port-forward svc/loki 3100:3100 -n default
```

---

## 🎓 What You’ve Mastered

```
✅ Prometheus retention
✅ Grafana backups
✅ Loki logs
✅ Troubleshooting
✅ OpenTelemetry concepts
✅ Production mindset
```

---

## 💡 Final Thought

```
Monitoring is not about dashboards.
Monitoring is about confidence.
```
