# Docker Observability Stack

A complete observability solution using Prometheus, Grafana, and Loki for monitoring both Docker containers and external systems.

## Components

- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **Loki**: Log aggregation and storage
- **Promtail**: Log collection agent with syslog receiver
- **cAdvisor**: Container metrics
- **Node Exporter**: Host machine metrics

## Quick Start

1. Start the stack:
```bash
docker-compose up -d
```

2. Access the services:
- Grafana: http://localhost:3000 (admin/admin)
- Prometheus: http://localhost:9090
- Loki: http://localhost:3100

## Configuration

### Monitoring Docker Containers

Containers are automatically discovered. To expose metrics from your containers:

1. Add these labels to your container:
```yaml
labels:
  - "prometheus.scrape=true"
  - "prometheus.port=8080"  # Your metrics port
  - "prometheus.path=/metrics"  # Optional, defaults to /metrics
```

### Monitoring External Systems

1. **For Prometheus metrics**, add targets to `prometheus/prometheus.yml`:
```yaml
- job_name: 'external-app'
  static_configs:
    - targets: ['app1.example.com:9090']
```

Or create a file in `prometheus/targets/`:
```yaml
- targets:
  - 'app1.example.com:9090'
  - 'app2.example.com:8080'
  labels:
    environment: 'production'
```

2. **For logs**, configure your external systems to send syslog to:
   - Protocol: TCP
   - Host: `<your-docker-host>`
   - Port: 514

## Adding Custom Dashboards

Place dashboard JSON files in `grafana/provisioning/dashboards/`

## Volumes

- `prometheus_data`: Prometheus time-series data
- `grafana_data`: Grafana dashboards and settings
- `loki_data`: Log storage

## Ports

- 3000: Grafana
- 9090: Prometheus
- 3100: Loki
- 514: Syslog receiver (TCP)
- 9100: Node Exporter
- 8080: cAdvisor