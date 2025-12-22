# Grafana & AlertManager with Docker Compose - Minimal Reference Guide

This guide demonstrates how to set up Grafana, Prometheus, and AlertManager using Docker Compose, and trigger a simple alert.

## Prerequisites

- Docker and Docker Compose installed
- Basic terminal/command line knowledge

## Project Structure

Create a directory for this project and the following structure:

```
alerting-demo/
├── docker-compose.yml
├── prometheus/
│   └── prometheus.yml
└── alertmanager/
    └── alertmanager.yml
```

## Step 1: Create docker-compose.yml

Create a file named `docker-compose.yml`:

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    networks:
      - monitoring

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_SECURITY_ADMIN_USER=admin
    networks:
      - monitoring

networks:
  monitoring:
    driver: bridge
```

## Step 2: Create Prometheus Configuration

Create `prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

rule_files:
  - 'alert_rules.yml'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

## Step 3: Create Alert Rules

Create `prometheus/alert_rules.yml`:

```yaml
groups:
  - name: example_alerts
    interval: 10s
    rules:
      - alert: PrometheusUp
        expr: up{job="prometheus"} == 1
        for: 10s
        labels:
          severity: info
        annotations:
          summary: "Prometheus is up and running"
          description: "This is a demo alert that fires when Prometheus is healthy."
      
      - alert: PrometheusDown
        expr: up{job="prometheus"} == 0
        for: 10s
        labels:
          severity: critical
        annotations:
          summary: "Prometheus is down"
          description: "Prometheus has been down for more than 10 seconds."
```

## Step 4: Create AlertManager Configuration

Create `alertmanager/alertmanager.yml`:

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default-receiver'
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h

receivers:
  - name: 'default-receiver'
    webhook_configs:
      - url: 'http://example.com/webhook'
        send_resolved: true
```

**Note**: This uses a dummy webhook URL. In production, you'd configure email, Slack, PagerDuty, etc.

## Step 5: Update Docker Compose for Alert Rules

Update the `docker-compose.yml` prometheus service to mount the alert rules:

```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    networks:
      - monitoring
```

## Step 6: Start the Stack

From your project directory, run:

```bash
docker-compose up -d
```

Check that all containers are running:

```bash
docker-compose ps
```

## Step 7: Access the Services

- **Prometheus**: http://localhost:9090
- **AlertManager**: http://localhost:9093
- **Grafana**: http://localhost:3000 (username: `admin`, password: `admin`)

## Step 8: Verify Alert is Firing

1. Open Prometheus at http://localhost:9090
2. Go to **Status** → **Alerts**
3. You should see the `PrometheusUp` alert in **Firing** state (since Prometheus is running)
4. Open AlertManager at http://localhost:9093
5. You should see the alert listed under **Alerts**

## Step 9: Configure Grafana (Optional)

1. Log into Grafana at http://localhost:3000
2. Go to **Connections** → **Data sources** → **Add data source**
3. Select **Prometheus**
4. Set URL to `http://prometheus:9090`
5. Click **Save & test**

### Add AlertManager Data Source

1. Go to **Connections** → **Data sources** → **Add data source**
2. Select **Alertmanager**
3. Set URL to `http://alertmanager:9093`
4. Set Implementation to **Prometheus**
5. Click **Save & test**

## Step 10: Trigger the Critical Alert

To trigger the `PrometheusDown` alert, stop the Prometheus container:

```bash
docker-compose stop prometheus
```

Wait 10-15 seconds, then check AlertManager at http://localhost:9093 - you won't see the alert because Prometheus is down (which is the ironic part of this demo).

Instead, let's create a better demo alert. Restart Prometheus:

```bash
docker-compose start prometheus
```

## Better Demo: Always-Firing Alert

Add this to `prometheus/alert_rules.yml`:

```yaml
      - alert: DemoAlertAlwaysFiring
        expr: vector(1)
        for: 10s
        labels:
          severity: warning
        annotations:
          summary: "Demo Alert - Always Firing"
          description: "This is a demo alert that always fires for testing purposes."
```

Reload Prometheus configuration:

```bash
docker-compose restart prometheus
```

After 10 seconds, check Prometheus alerts and AlertManager - you'll see the new alert firing.

## Cleanup

To stop and remove all containers:

```bash
docker-compose down
```

To remove volumes as well:

```bash
docker-compose down -v
```

## Summary

You now have:
- ✅ Prometheus collecting metrics
- ✅ AlertManager receiving alerts
- ✅ Grafana for visualization
- ✅ A working alert that fires automatically
- ✅ Understanding of how alerts flow from Prometheus → AlertManager

## Next Steps

- Configure real notification channels (email, Slack, etc.) in `alertmanager.yml`
- Create custom dashboards in Grafana
- Add more realistic alerting rules based on actual metrics
- Explore Grafana's built-in alerting (alternative to Prometheus alerts)
