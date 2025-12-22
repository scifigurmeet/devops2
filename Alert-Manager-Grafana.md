
## 🎯 What you’ll achieve in 15–20 minutes

✔ Run **Prometheus + Alertmanager + Grafana** with Docker Compose
✔ Create a **Prometheus alerting rule**
✔ Route alerts via **Alertmanager**
✔ View & manage alerts in **Grafana UI**

---

## 🧠 Components (Very Short)

* **Prometheus** – collects metrics & evaluates alert rules
* **Alertmanager** – routes & manages alerts
* **Grafana** – visualizes metrics & alerts

---

## 📁 Project Structure

```text
alerting-demo/
│
├── docker-compose.yml
├── prometheus.yml
├── alert.rules.yml
└── alertmanager.yml
```

---

## 1️⃣ Docker Compose (All-in-One)

**docker-compose.yml**

```yaml
version: "3.8"

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alert.rules.yml:/etc/prometheus/alert.rules.yml
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
    ports:
      - "9090:9090"

  alertmanager:
    image: prom/alertmanager
    container_name: alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - "--config.file=/etc/alertmanager/alertmanager.yml"
    ports:
      - "9093:9093"

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
```

---

## 2️⃣ Prometheus Configuration

**prometheus.yml**

```yaml
global:
  scrape_interval: 15s

rule_files:
  - "alert.rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["prometheus:9090"]
```

---

## 3️⃣ Alerting Rule (Very Simple)

**alert.rules.yml**

```yaml
groups:
- name: demo-alerts
  rules:
  - alert: PrometheusDown
    expr: up == 0
    for: 10s
    labels:
      severity: critical
    annotations:
      summary: "Prometheus is DOWN"
      description: "Prometheus target is not reachable"
```

📌 This fires if any scraped target goes **down for 10 seconds**.

---

## 4️⃣ Alertmanager Configuration

**alertmanager.yml**

```yaml
global:
  resolve_timeout: 1m

route:
  receiver: "default-receiver"

receivers:
- name: "default-receiver"
  webhook_configs:
    - url: "http://localhost:5001"
```

⚠️ No real webhook needed for demo — Alertmanager UI will still show alerts.

---

## 5️⃣ Start the Stack

From project directory:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

---

## 6️⃣ Verify Everything (Important URLs)

| Tool         | URL                                            |
| ------------ | ---------------------------------------------- |
| Prometheus   | [http://localhost:9090](http://localhost:9090) |
| Alertmanager | [http://localhost:9093](http://localhost:9093) |
| Grafana      | [http://localhost:3000](http://localhost:3000) |

Grafana login:

```
user: admin
pass: admin
```

---

## 7️⃣ Trigger an Alert (Demo)

Stop Prometheus container:

```bash
docker stop prometheus
```

⏱ After ~10 seconds:

* **Alertmanager UI** → shows `PrometheusDown`
* **Grafana** → alert appears automatically

Restart:

```bash
docker start prometheus
```

Alert resolves ✅

---

## 8️⃣ View Alerts in Grafana

1. Open **Grafana**
2. Go to **Alerting → Alert rules**
3. You’ll see **Prometheus alerts synced**
4. Click → Inspect → Status, labels, annotations

📌 Grafana reads alerts directly from Prometheus.

---

## 🧩 How Alert Flow Works (Mental Model)

```text
Metrics → Prometheus → Alert Rules → Alertmanager → Grafana UI
```

---

## ✅ What You Learned

✔ Prometheus alert rule basics
✔ Alertmanager routing concept
✔ Grafana alert visibility
✔ Docker Compose monitoring stack

---
