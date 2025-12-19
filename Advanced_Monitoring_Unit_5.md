
---

# 📘 Unit V – Advanced Monitoring & Visualization

**Using Prometheus, Alertmanager & Grafana**
*(Beginner → Industry Level | Windows + Minikube + EKS Concepts)*

---

## 🎯 What Students Will Learn in Unit V

By the end of this unit, students will be able to:

* Write **PromQL queries** confidently
* Understand **how alerting works**
* Configure **Alertmanager**
* Monitor **Node, Pod & Container metrics**
* Build **advanced Grafana dashboards**
* Use **variables in Grafana**
* Understand **EKS monitoring architecture**
* Apply **industry best practices**

---

## 🧠 First: Why “Advanced” Monitoring?

Unit IV taught us:

* **What to monitor**
* **How to collect metrics**
* **How to visualize them**

Unit V answers:

> **How do we query better, alert smarter, and scale monitoring like real companies?**

---

## 🧩 Unit V Monitoring Stack (Extended)

```
Kubernetes (Minikube / EKS)
        ↓
   Prometheus Server
        ↓
   Alertmanager
        ↓
Notifications (Slack, Email, etc.)

Prometheus
    ↓
 Grafana Dashboards
```

---

## 🧪 Part 1: Prometheus Query Language (PromQL)

---

## ❓ What is PromQL?

**PromQL** is the **query language of Prometheus**.

It helps us:

* Filter metrics
* Calculate rates
* Aggregate data
* Create alerts
* Build dashboards

👉 Think of PromQL like **SQL for metrics**

---

## 🔤 PromQL Basics (Very Important)

### 🔹 Metric Name

Example:

```promql
up
```

👉 Shows if targets are **up (1)** or **down (0)**

---

### 🔹 Labels (Filtering)

```promql
up{job="kubernetes-nodes"}
```

👉 Filters metrics using labels

---

## ➕ PromQL Operators

| Type       | Examples        |
| ---------- | --------------- |
| Arithmetic | `+ - * /`       |
| Comparison | `> < ==`        |
| Logical    | `and or unless` |

---

## 📈 Rate & Time-Based Queries

### CPU Usage (Most Common)

```promql
rate(container_cpu_usage_seconds_total[1m])
```

🧠 Meaning:

* CPU usage
* Calculated per second
* Over last 1 minute

---

### Total CPU Usage (Cluster Level)

```promql
sum(rate(container_cpu_usage_seconds_total[1m]))
```

---

### Memory Usage (MB)

```promql
container_memory_usage_bytes / (1024 * 1024)
```

---

## 🧠 Aggregation Functions

| Function  | Purpose |
| --------- | ------- |
| `sum()`   | Total   |
| `avg()`   | Average |
| `max()`   | Maximum |
| `min()`   | Minimum |
| `count()` | Count   |

Example:

```promql
avg(container_memory_usage_bytes)
```

---

## 🚨 Part 2: Alerting Concepts (Very Important)

---

## ❓ Why Alerts?

Dashboards are **reactive**
Alerts are **proactive**

👉 Alerts tell us **before users complain**

---

## 🧠 Real-Life Analogy

| Scenario     | Alert           |
| ------------ | --------------- |
| CPU too high | 🔔 Alarm        |
| Pod crashed  | 🚨 Notification |
| Memory leak  | ⚠️ Warning      |

---

## 🧱 Alerting Components

```
Prometheus → Alert Rules → Alertmanager → Notification
```

---

## ⚙️ What is Alertmanager?

**Alertmanager**:

* Receives alerts from Prometheus
* Groups & filters alerts
* Sends notifications

Supports:

* Email
* Slack
* PagerDuty
* Webhooks

---

## 🧪 Part 3: Alertmanager Setup (Minikube)

> Prometheus Helm chart already installs Alertmanager ✅

---

### 🔍 Verify Alertmanager Pod

```cmd
kubectl get pods
```

You should see:

```
alertmanager-prometheus-alertmanager
```

---

### 🌐 Access Alertmanager UI

```cmd
kubectl port-forward svc/prometheus-alertmanager 9093:80
```

Open:

```
http://localhost:9093
```

---

## 📝 Writing Alerting Rules (Simple)

### Example: High CPU Alert

```yaml
groups:
- name: cpu-alerts
  rules:
  - alert: HighCPUUsage
    expr: sum(rate(container_cpu_usage_seconds_total[1m])) > 0.8
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
```

🧠 Meaning:

* CPU > 80%
* For 1 minute
* Then trigger alert

---

## 📊 Part 4: Monitoring Node, Pod & Container Metrics

---

## 🖥️ Node Monitoring

### Available Node Memory

```promql
node_memory_MemAvailable_bytes / (1024 * 1024)
```

---

### Node CPU Usage

```promql
rate(node_cpu_seconds_total[1m])
```

---

## 📦 Pod Monitoring

### Pod Status

```promql
kube_pod_status_phase
```

---

### Pod Restart Count

```promql
kube_pod_container_status_restarts_total
```

---

## 📦 Container Monitoring

### Container CPU

```promql
rate(container_cpu_usage_seconds_total[1m])
```

---

### Container Memory

```promql
container_memory_usage_bytes
```

---

## ☁️ Part 5: Prometheus & Grafana in EKS (Conceptual)

---

## 🧠 How Monitoring Works in EKS

```
EKS Cluster
  ├── Control Plane (AWS Managed)
  ├── Worker Nodes
  └── Applications
        ↓
     Prometheus
        ↓
      Grafana
```

---

## 🔑 Key Differences (Minikube vs EKS)

| Feature      | Minikube | EKS        |
| ------------ | -------- | ---------- |
| Cluster      | Local    | AWS        |
| Storage      | Local    | EBS        |
| LoadBalancer | Tunnel   | AWS ELB    |
| Scale        | Limited  | Production |

👉 **Monitoring concepts remain SAME**

---

## 📈 Part 6: Advanced Grafana Dashboards

---

## 🎯 Dashboards to Build

* CPU Usage
* Memory Usage
* Network Traffic
* Pod Health
* Node Health

---

## 📊 CPU Dashboard Query

```promql
sum(rate(container_cpu_usage_seconds_total[1m])) by (pod)
```

---

## 💾 Memory Dashboard Query

```promql
sum(container_memory_usage_bytes) by (pod) / (1024 * 1024)
```

---

## 🌐 Network Usage

```promql
rate(container_network_receive_bytes_total[1m])
```

---

## 🔁 Part 7: Grafana Variables (Very Important)

---

## ❓ Why Variables?

Instead of creating **100 dashboards**:
👉 Use **1 dashboard + variables**

---

## 🧩 Example Variable

Variable Name: `pod`

Query:

```promql
label_values(kube_pod_info, pod)
```

Usage in panel:

```promql
container_cpu_usage_seconds_total{pod="$pod"}
```

🎯 Now dashboard is **dynamic**

---

## 🎨 Part 8: Visualization Best Practices

---

### ✅ Do This

* Use clear titles
* Label units (MB, %, ms)
* Use correct graph type
* Limit time range
* Group related panels

---

### ❌ Avoid This

* Too many panels
* Mixing unrelated metrics
* No legends
* No alerts

---

## 🏭 Part 9: Industry Case Studies (Conceptual)

---

## 🏢 Case Study 1: E-Commerce Platform

**Monitored Metrics**

* API latency
* Pod restarts
* CPU spikes during sale

**Outcome**

* Auto-scale pods
* Alert before crash
* Faster incident response

---

## 🏦 Case Study 2: Banking Application

**Monitored Metrics**

* Memory leaks
* Node health
* Error rate

**Outcome**

* Zero downtime
* Regulatory compliance
* Faster debugging

---

## 🧠 Final Mental Model (Teach This)

* **PromQL** → Ask questions
* **Prometheus** → Collect answers
* **Alertmanager** → Warn humans
* **Grafana** → Show stories

---

## ✅ Unit V Completed ✔️

Students now understand:

✔ PromQL
✔ Alerting & Alertmanager
✔ Node, Pod, Container Monitoring
✔ Advanced Grafana Dashboards
✔ Variables & Best Practices
✔ Real-world Cloud Monitoring

---
