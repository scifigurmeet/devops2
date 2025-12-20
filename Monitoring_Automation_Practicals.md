# 🚀 Unit VI – Automation & Future Trends: Practical Guide

**For Absolute Beginners | Hands-On Practice**

---

## 📋 Prerequisites Check

Before starting, verify your setup:

```bash
# Check Minikube is running
minikube status

# Check Prometheus pods
kubectl get pods -n default | grep prometheus

# Check Grafana pods
kubectl get pods -n default | grep grafana
```

Expected: All pods should show `Running` status ✅

---

## 🎯 What We'll Practice Today

1. ✅ **Backup & Restore Grafana Dashboards** (Export/Import)
2. ✅ **Configure Prometheus Data Retention** (Storage management)
3. ✅ **Basic Security Setup** (Authentication & HTTPS concepts)
4. ✅ **Install & Explore Grafana Loki** (Log aggregation)
5. ✅ **Troubleshoot Common Issues** (Real debugging)
6. ✅ **Understand OpenTelemetry** (Future standard)

---

## 📦 Part 1: Understanding Your Current Setup with Helm

### 🧠 What is Helm? (Quick Recap)

Helm is like **App Store for Kubernetes**:
- Instead of creating 10+ YAML files manually
- Helm installs everything with 1 command
- You already used it to install Prometheus/Grafana!

### 🔍 Let's See What Helm Installed

```bash
# List all Helm releases
helm list -A

# See what's in your Prometheus release
helm get values prometheus -n default

# See all resources created by Helm
helm get manifest prometheus -n default
```

**What you'll see**: Deployments, Services, ConfigMaps all created together!

---

## 💾 Part 2: Backup & Restore Grafana Dashboards

### 🧠 Why Backup?

Imagine you spent 2 hours creating a beautiful dashboard... then accidentally deleted it! 😱

**Solution**: Regular backups!

---

### 📤 Method 1: Manual Backup (via UI)

**Step 1**: Access Grafana

```bash
# Get Grafana URL
minikube service grafana --url

# Or port-forward
kubectl port-forward svc/grafana 3000:80 -n default
```

Open: `http://localhost:3000`

**Step 2**: Export Dashboard
1. Login to Grafana (default: admin/admin)
2. Open any dashboard
3. Click ⚙️ (Settings) → JSON Model
4. Copy the entire JSON
5. Save to a file: `dashboard-backup.json`

---

### 📥 Restore Dashboard

**Step 1**: Go to Grafana UI
**Step 2**: Click `+` → Import
**Step 3**: Paste JSON or upload file
**Step 4**: Click **Load** → **Import**

✅ Dashboard restored!

---

### 📤 Method 2: Automated Backup (Command Line)

**Step 1**: Get Grafana admin password

```bash
# Find Grafana secret
kubectl get secret grafana -n default -o jsonpath="{.data.admin-password}" | base64 --decode
echo
```

**Step 2**: Create backup script

```bash
# Create backup directory
mkdir grafana-backups
cd grafana-backups

# Get Grafana URL
GRAFANA_URL="http://localhost:3000"
GRAFANA_USER="admin"
GRAFANA_PASS="your-password-here"

# List all dashboards (you'll need curl or a REST client)
# This is a preview - in production, teams use automation tools
```

**Note**: For beginners, UI method is easier. In industry, teams use tools like:
- Grafana API scripts
- Git repositories (store dashboards as code)
- Backup operators

---

### 🧪 Practice Exercise

1. Create a simple dashboard in Grafana
2. Add a panel showing `up` metric
3. Export it as JSON
4. Delete the dashboard
5. Import it back

**You've mastered backup/restore!** 🎉

---

## 🗂️ Part 3: Managing Prometheus Data Retention

### 🧠 What is Data Retention?

**Simple explanation**: How long Prometheus keeps your metrics

Example:
- **7 days retention** → Metrics older than 7 days are deleted
- **30 days retention** → Metrics kept for 30 days

---

### ⚖️ Why It Matters

| Short Retention (7 days) | Long Retention (90 days) |
|-------------------------|--------------------------|
| ✅ Less disk space       | ❌ More disk space       |
| ✅ Faster queries        | ❌ Slower queries        |
| ❌ Less history          | ✅ More analysis         |

---

### 🔍 Check Current Retention

```bash
# Check Prometheus configuration
kubectl get configmap prometheus-server -n default -o yaml | grep retention
```

---

### ⚙️ Configure Retention (Practical)

**Step 1**: Check current Prometheus deployment

```bash
kubectl get deployment prometheus-server -n default -o yaml | grep -A 5 "args:"
```

You'll see something like:
```yaml
args:
  - --storage.tsdb.retention.time=15d
  - --config.file=/etc/prometheus/prometheus.yml
```

**Step 2**: Update retention using Helm

```bash
# Update to 7 days retention
helm upgrade prometheus prometheus-community/prometheus \
  --set server.retention=7d \
  -n default

# Or 30 days
helm upgrade prometheus prometheus-community/prometheus \
  --set server.retention=30d \
  -n default
```

**Step 3**: Verify the change

```bash
kubectl get deployment prometheus-server -n default -o yaml | grep retention
```

---

### 🧮 Calculate Storage Needed

**Simple formula**:
```
Storage = Metrics per second × Retention days × 24 hours × 3600 seconds × ~2 bytes
```

**Example**:
- 10,000 metrics/sec
- 15 days retention
- ≈ **26 GB** needed

---

### 🧪 Practice Exercise

1. Check your current retention
2. Change it to 3 days (for testing)
3. Verify the change
4. Change it back to 15 days

---

## 🔐 Part 4: Securing Monitoring Tools

### 🧠 Security 3 Pillars

```
Authentication → Who are you?
Authorization  → What can you do?
Encryption     → Protect data in transit
```

---

### 🔑 Practice 1: Change Grafana Admin Password

**Step 1**: Access Grafana pod

```bash
# Get Grafana pod name
kubectl get pods -n default | grep grafana

# Example: grafana-5c7d4f8b6d-xyz12
POD_NAME="grafana-xxxxx"

# Access the pod
kubectl exec -it $POD_NAME -n default -- /bin/sh
```

**Step 2**: Inside the pod, use Grafana CLI

```bash
# Change admin password
grafana-cli admin reset-admin-password NewSecurePassword123

# Exit pod
exit
```

**Step 3**: Test new password in browser

---

### 🛂 Practice 2: Create Different User Roles

**Step 1**: Login to Grafana as admin

**Step 2**: Create users with different roles

1. Go to ⚙️ (Configuration) → Users
2. Click **Invite**
3. Add email: `viewer@example.com`
4. Role: **Viewer** (can only view dashboards)
5. Click **Submit**

**Roles explained**:
- **Viewer** → Read-only access
- **Editor** → Can edit dashboards
- **Admin** → Full control

---

### 🔐 Practice 3: Enable HTTPS (Concept)

**In production, you'd use**:
```bash
# Create TLS certificate (example - don't run in Minikube)
kubectl create secret tls grafana-tls \
  --cert=path/to/cert.pem \
  --key=path/to/key.pem \
  -n default
```

**Then configure Grafana to use it**

**For Minikube**: Use port-forward with HTTPS is complex. In real environments:
- Use **Ingress** with TLS
- Use **cert-manager** for automatic certificates
- Use cloud load balancers with SSL

---

### 🧪 Security Best Practices (Checklist)

```
✅ Change default passwords
✅ Use strong passwords (12+ characters)
✅ Enable HTTPS in production
✅ Use role-based access (RBAC)
✅ Regularly update credentials
✅ Audit access logs
```

---

## 📜 Part 5: Introduction to Grafana Loki

### 🧠 What is Grafana Loki?

**Simple explanation**: Loki is like Prometheus, but for **logs instead of metrics**

| Tool       | What it stores | Example                    |
|------------|----------------|----------------------------|
| Prometheus | Numbers        | CPU: 80%, Memory: 2GB      |
| Loki       | Text logs      | "Error: Connection failed" |

---

### 🎯 Why Do We Need Loki?

**Problem**: You see CPU is high in Prometheus, but **why?**

**Solution**: Check logs in Loki!

```
Prometheus says: CPU = 95%
         ↓
Loki shows: "OutOfMemoryError in app.jar"
         ↓
You know the problem!
```

---

### 🧩 How Loki Works

```
Application writes logs
         ↓
Promtail collects logs (like a log agent)
         ↓
Loki stores logs (efficiently)
         ↓
Grafana displays logs (same UI!)
```

---

### 🚀 Install Grafana Loki (Practical)

**Step 1**: Add Loki Helm repository

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

**Step 2**: Install Loki stack (Loki + Promtail)

```bash
# Install Loki with minimal resources (for Minikube)
helm install loki grafana/loki-stack \
  --set loki.persistence.enabled=false \
  --set promtail.enabled=true \
  -n default
```

**What this installs**:
- **Loki**: Log storage system
- **Promtail**: Log collector (runs on each node)

---

**Step 3**: Verify installation

```bash
# Check Loki pods
kubectl get pods -n default | grep loki

# You should see:
# loki-0                    1/1     Running
# loki-promtail-xxxxx       1/1     Running
```

---

### 🔌 Connect Loki to Grafana

**Step 1**: Access Grafana UI

```bash
kubectl port-forward svc/grafana 3000:80 -n default
```

**Step 2**: Add Loki as data source

1. Login to Grafana
2. Go to ⚙️ (Configuration) → Data Sources
3. Click **Add data source**
4. Select **Loki**
5. Enter URL: `http://loki:3100`
6. Click **Save & Test**

✅ You should see "Data source is working"

---

### 🔍 Query Logs in Grafana

**Step 1**: Go to **Explore** (compass icon 🧭)

**Step 2**: Select **Loki** data source

**Step 3**: Try these queries:

```logql
# Show all logs
{namespace="default"}

# Show logs from specific pod
{pod="prometheus-server-xxxxx"}

# Show error logs only
{namespace="default"} |= "error"

# Show logs with "failed"
{namespace="default"} |= "failed"
```

---

### 🧪 Generate Some Logs (Testing)

```bash
# Create a test pod that generates logs
kubectl run log-generator \
  --image=busybox \
  --restart=Never \
  -- /bin/sh -c "while true; do echo 'Log message at $(date)'; sleep 5; done"

# View logs
kubectl logs -f log-generator

# Now check these logs in Grafana Loki!
# Query: {pod="log-generator"}
```

---

### 📊 Create Log Dashboard

**Step 1**: In Grafana, create new dashboard

**Step 2**: Add panel

**Step 3**: Select Loki data source

**Step 4**: Use query:
```logql
sum(count_over_time({namespace="default"}[1m]))
```

This shows: **Logs per minute**

---

### 🎓 Loki Summary (What You Learned)

```
✅ Loki stores logs
✅ Promtail collects logs from Kubernetes
✅ Grafana displays both metrics AND logs
✅ You can correlate: "CPU high" → "Check logs" → "Find error"
```

---

## 🔭 Part 6: Understanding OpenTelemetry

### 🧠 What is OpenTelemetry (OTel)?

**The problem it solves**:

Before:
```
App → Prometheus (metrics)
App → Loki (logs)
App → Jaeger (traces)
App → DataDog (monitoring)
```

Each tool needs **different code**! 😫

---

**After OpenTelemetry**:

```
App → OpenTelemetry → Any monitoring tool
```

**One standard, works everywhere!**

---

### 🔺 The 3 Pillars of Observability

| Pillar  | What it shows | Example                           |
|---------|---------------|-----------------------------------|
| Metrics | Numbers       | CPU: 80%                          |
| Logs    | Messages      | "Error: database timeout"         |
| Traces  | Request flow  | API → Service A → Service B → DB |

OpenTelemetry handles **all three**!

---

### 🎯 Why OpenTelemetry Matters (Real Example)

**Before OTel**:
```python
# Different library for each tool
import prometheus_client  # For metrics
import logging           # For logs
import jaeger_client     # For traces
```

**With OTel**:
```python
# One library for everything
from opentelemetry import trace, metrics, logs
```

---

### 🚀 OpenTelemetry Demo (Conceptual)

**How it works**:

```
Your Application
      ↓
OpenTelemetry SDK (installed in app)
      ↓
OpenTelemetry Collector (receives data)
      ↓
Sends to: Prometheus, Loki, Jaeger, etc.
```

---

### 🧪 Install OpenTelemetry Collector (Optional Practice)

```bash
# Add OpenTelemetry Helm repo
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Install collector (minimal setup)
helm install otel-collector open-telemetry/opentelemetry-collector \
  --set mode=deployment \
  -n default
```

**Note**: This is advanced. For beginners, understanding the concept is enough!

---

### 🎓 OpenTelemetry Summary

```
✅ One standard for metrics, logs, traces
✅ Write instrumentation code once
✅ Send data to multiple tools
✅ Future of observability
✅ Supported by: AWS, Google Cloud, Azure, Prometheus, Grafana
```

---

## 🛠️ Part 7: Troubleshooting Common Issues

### ❌ Issue 1: No Data in Grafana

**Symptoms**: Dashboard shows "No Data"

**Debug Steps**:

```bash
# Step 1: Check Prometheus is running
kubectl get pods -n default | grep prometheus

# Step 2: Check Prometheus targets
kubectl port-forward svc/prometheus-server 9090:80 -n default
# Open: http://localhost:9090/targets
# All targets should be "UP"

# Step 3: Test query in Prometheus
# Open: http://localhost:9090/graph
# Run: up
# Should show results!

# Step 4: Check Grafana data source
# Grafana → Data Sources → Prometheus → Test
```

---

### ❌ Issue 2: Prometheus Pod Crashing

**Symptoms**: Prometheus pod restarts frequently

**Debug Steps**:

```bash
# Check pod status
kubectl get pods -n default | grep prometheus

# Check logs
kubectl logs prometheus-server-xxxxx -n default

# Common causes:
# 1. Out of memory
# 2. Invalid configuration
# 3. Disk full
```

**Solution 1: Increase memory**

```bash
helm upgrade prometheus prometheus-community/prometheus \
  --set server.resources.limits.memory=2Gi \
  --set server.resources.requests.memory=1Gi \
  -n default
```

---

### ❌ Issue 3: Grafana Dashboard Slow

**Symptoms**: Dashboard takes 30+ seconds to load

**Causes**:
1. Query time range too large (30 days)
2. Too many metrics in one query
3. Missing indexes

**Solutions**:

```bash
# Solution 1: Reduce time range
# Change from "Last 30 days" → "Last 6 hours"

# Solution 2: Use aggregation
# Instead of: rate(http_requests_total[5m])
# Use: avg(rate(http_requests_total[5m])) by (job)

# Solution 3: Add more resources to Prometheus
helm upgrade prometheus prometheus-community/prometheus \
  --set server.resources.limits.cpu=2 \
  -n default
```

---

### ❌ Issue 4: Loki Not Showing Logs

**Debug Steps**:

```bash
# Check Promtail is running
kubectl get pods -n default | grep promtail

# Check Promtail logs
kubectl logs loki-promtail-xxxxx -n default

# Check Loki is reachable
kubectl port-forward svc/loki 3100:3100 -n default
curl http://localhost:3100/ready

# Test Loki query
# Grafana → Explore → Loki → {namespace="default"}
```

---

### 🔧 Troubleshooting Checklist

```
✅ Check pod status: kubectl get pods
✅ Check pod logs: kubectl logs <pod-name>
✅ Check service endpoints: kubectl get svc
✅ Check configurations: kubectl get configmap
✅ Test connectivity: kubectl port-forward
✅ Verify resources: kubectl top pods
```

---

## 🌐 Part 8: AWS CloudWatch Integration (Concept)

### 🧠 What is AWS CloudWatch?

**CloudWatch** = AWS's monitoring service

When your Kubernetes runs on **AWS EKS**, CloudWatch automatically collects:
- EC2 instance metrics
- EKS cluster metrics
- Application logs

---

### 🔗 How It Connects

```
EKS Cluster
    ↓
CloudWatch (AWS metrics)
    ↓
Prometheus (scrapes CloudWatch Exporter)
    ↓
Grafana (displays everything)
```

---

### 🆚 CloudWatch vs Prometheus

| Feature          | CloudWatch      | Prometheus  |
|------------------|-----------------|-------------|
| Where           | AWS only        | Anywhere    |
| Cost            | Pay per metric  | Free/Open   |
| Integration     | Deep AWS        | K8s native  |
| Best for        | AWS resources   | Applications|

**In practice**: Use **both together**!

---

### 🎓 Summary

```
✅ CloudWatch = AWS monitoring
✅ Best for AWS infrastructure
✅ Can export to Prometheus
✅ Combine with Grafana for unified view
```

---

## 🔄 Part 9: End-to-End Monitoring Pipeline

### 🎨 Complete Picture (What You Built)

```
┌─────────────────────────────────────────────────────┐
│                  Your Application                    │
└───────────┬─────────────────────────────────────────┘
            │
            ├──→ Metrics ──→ Prometheus
            │                    ↓
            │              (stores numbers)
            │                    ↓
            │               Alertmanager
            │                    ↓
            │              (sends alerts)
            │
            └──→ Logs ────→ Promtail ──→ Loki
                                         ↓
                                    (stores logs)
                                         ↓
                    ┌────────────────────┴──────────────────┐
                    │                                       │
                    ▼                                       ▼
              Grafana Dashboards                      Alert Notifications
              (metrics + logs)                        (email, Slack)
                    │
                    ▼
               Your Team 👥
```

---

### 🎯 What Each Tool Does

```
Application      → Generates metrics & logs
Prometheus       → Collects & stores metrics
Loki             → Collects & stores logs
Promtail         → Ships logs to Loki
Alertmanager     → Sends alerts
Grafana          → Visualizes everything
```

---

### 🚀 Industry Best Practices

```
✅ Use Helm for installation
✅ Store configs in Git
✅ Enable authentication
✅ Set up HTTPS in production
✅ Backup dashboards regularly
✅ Monitor the monitoring system itself!
✅ Set reasonable retention periods
✅ Use labels for organization
✅ Create runbooks for alerts
```

---

## 📝 Part 10: Final Practice Exercises

### Exercise 1: Complete Monitoring Setup

```bash
# 1. Check all monitoring components
kubectl get pods -n default

# 2. Access each service
kubectl port-forward svc/prometheus-server 9090:80 -n default
kubectl port-forward svc/grafana 3000:80 -n default
kubectl port-forward svc/loki 3100:3100 -n default

# 3. Verify data flow
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3000
# Loki: http://localhost:3100/ready
```

---

### Exercise 2: Create End-to-End Dashboard

**Goal**: One dashboard showing metrics AND logs

1. Create new dashboard in Grafana
2. Add panel: Prometheus metrics (CPU usage)
3. Add panel: Loki logs (from same pod)
4. Save dashboard
5. Export as JSON (backup!)

---

### Exercise 3: Simulate Issue & Debug

```bash
# Create a failing pod
kubectl run failing-app \
  --image=busybox \
  --restart=Never \
  -- /bin/sh -c "echo 'Starting app'; sleep 5; echo 'ERROR: Database connection failed'; exit 1"

# Now debug using your monitoring:
# 1. Check Prometheus: Is pod up?
# 2. Check Loki: What error did it show?
# 3. Check Grafana: Visualize the failure
```

---

## 🎓 What You've Mastered

```
✅ Backup & restore Grafana dashboards
✅ Configure Prometheus data retention
✅ Basic security (passwords, roles)
✅ Install & use Grafana Loki
✅ Understand logs vs metrics vs traces
✅ Troubleshoot monitoring issues
✅ Understand OpenTelemetry (future standard)
✅ Know AWS CloudWatch integration
✅ Built complete monitoring pipeline
```

---

## 🚀 Next Steps (For Your Career)

### Beginner → Intermediate

```
1. Learn PromQL deeply (query language)
2. Create custom exporters
3. Set up Alertmanager with real notifications
4. Implement distributed tracing (Jaeger)
5. Use Thanos for long-term storage
```

### Tools to Explore

```
- Thanos (Prometheus at scale)
- Cortex (Multi-tenant Prometheus)
- Tempo (Distributed tracing)
- VictoriaMetrics (Fast metrics DB)
- Mimir (Grafana's metrics backend)
```

---

## 📚 Useful Commands Cheat Sheet

```bash
# Helm operations
helm list -A
helm upgrade <release> <chart>
helm rollback <release>

# Kubernetes debugging
kubectl get pods -n <namespace>
kubectl logs <pod-name>
kubectl describe pod <pod-name>
kubectl port-forward svc/<service> <local-port>:<remote-port>

# Prometheus
curl http://localhost:9090/api/v1/targets
curl http://localhost:9090/api/v1/query?query=up

# Loki
curl http://localhost:3100/ready
curl http://localhost:3100/metrics
```

---

## 🎉 Congratulations!

You now understand:
- **Modern monitoring architecture**
- **Industry-standard tools**
- **Practical troubleshooting**
- **Future trends in observability**

**You're ready for real DevOps/SRE work!** 🚀

---

## 💡 Remember

```
"Monitoring is not just about collecting data.
 It's about understanding your system
 and fixing problems before users notice them."
```

**Keep practicing, keep learning!** 🎯
