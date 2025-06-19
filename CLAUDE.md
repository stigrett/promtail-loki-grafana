# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Start the monitoring stack:**
```bash
docker-compose up -d
```

**Stop the monitoring stack:**
```bash
docker-compose down
```

**View logs:**
```bash
docker-compose logs -f [service_name]
# Service names: rsyslog, loki, promtail, prometheus, node-exporter, cadvisor, grafana
```

**Restart a specific service:**
```bash
docker-compose restart [service_name]
```

**Check service status:**
```bash
docker-compose ps
```

**View real-time logs from all services:**
```bash
docker-compose logs -f
```

## Architecture

This repository implements a complete observability stack using Docker Compose with the following components:

- **rsyslog** (port 514 UDP/TCP): Network syslog server that receives logs from remote systems
- **Loki** (port 3100): Log aggregation system that stores and queries logs from Promtail
- **Promtail**: Log shipping agent that reads logs from both local system and rsyslog output
- **Prometheus** (port 9090): Metrics collection and storage, configured to scrape Node Exporter and cAdvisor
- **Node Exporter** (port 9100): Exposes host-level metrics for Prometheus
- **cAdvisor** (port 18080, internal 8080): Container metrics collection for Docker containers
- **Grafana** (port 3000): Visualization platform for logs and metrics with provisioned datasources

### Service Communication
- rsyslog receives network syslog on port 514 (UDP/TCP) and writes to `/var/log/remote/`
- Promtail reads from rsyslog output and pushes to Loki at `http://loki:3100/loki/api/v1/push`
- Prometheus scrapes metrics from:
  - Node Exporter at `node-exporter:9100`
  - cAdvisor at `cadvisor:8080`
- Grafana uses provisioned datasources for Loki and Prometheus

### Configuration Files
- `rsyslog.conf`: rsyslog configuration for network syslog reception
- `loki-config.yml`: Loki server configuration with local storage and TSDB
- `promtail-config.yml`: Promtail scraping configuration for system logs and rsyslog output
- `prometheus.yml`: Prometheus scraping configuration for Node Exporter and cAdvisor
- `docker-compose.yml`: Service definitions and volume configuration
- `grafana-provisioning/`: Grafana provisioning files for datasources and dashboards

### Storage
All volumes use bind mounts to `/mnt/storage_pool/logs/`:
- `rsyslog-logs`: Shared between rsyslog and Promtail at `/mnt/storage_pool/logs/syslog`
- `loki-data`: Loki TSDB storage at `/mnt/storage_pool/logs/loki-data`
- `prometheus-data`: Prometheus metrics storage at `/mnt/storage_pool/logs/prometheus-data` (30-day retention)
- `grafana-data`: Grafana data at `/mnt/storage_pool/logs/grafana-data` (NFS-ready)
- `grafana-logs`: Grafana logs at `/mnt/storage_pool/logs/grafana-logs` (NFS-ready)

### Access Points
- **Grafana UI**: http://localhost:3000 (default: admin/admin)
- **Prometheus UI**: http://localhost:9090
- **Loki API**: http://localhost:3100
- **Node Exporter metrics**: http://localhost:9100/metrics
- **cAdvisor UI**: http://localhost:18080

### NFS Configuration
When your NFS server is ready, update the volume definitions in docker-compose.yml:
```yaml
grafana-data:
  driver: local
  driver_opts:
    type: nfs
    o: "addr=YOUR_NFS_SERVER,rw,nfsvers=4"
    device: ":/path/to/grafana/data"
```

### Important Notes
- **Service Dependencies**: Services are properly ordered with `depends_on` directives
- **Host Access**: Node Exporter and cAdvisor require host filesystem access for accurate metrics
- **Privileged Mode**: cAdvisor runs in privileged mode to access Docker statistics
- **Log Format**: rsyslog outputs in two formats - `syslog.log` for Promtail parsing and `debug.log` for troubleshooting
- **Security**: No authentication is configured on Loki/Prometheus endpoints (consider adding in production)