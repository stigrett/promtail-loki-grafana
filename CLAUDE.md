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
```

**Restart a specific service:**
```bash
docker-compose restart [service_name]
```

## Architecture

This repository implements a complete observability stack using Docker Compose with the following components:

- **rsyslog** (port 514 UDP/TCP): Network syslog server that receives logs from remote systems
- **Loki** (port 3100): Log aggregation system that stores and queries logs from Promtail
- **Promtail**: Log shipping agent that reads logs from both local system and rsyslog output
- **Prometheus** (port 9090): Metrics collection and storage, configured to scrape Node Exporter and cAdvisor
- **Node Exporter** (port 9100): Exposes host-level metrics for Prometheus
- **cAdvisor** (port 8080): Container metrics collection for Docker containers
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
- `loki-config.yml`: Loki server configuration with local storage (30-day retention recommended)
- `promtail-config.yml`: Promtail scraping configuration for system logs and rsyslog output
- `prometheus.yml`: Prometheus scraping configuration for Node Exporter and cAdvisor
- `docker-compose.yml`: Service definitions and volume configuration
- `grafana-provisioning/`: Grafana provisioning files for datasources and dashboards

### Storage
- Loki uses local volume storage for performance (configurable retention)
- Grafana data and logs use volumes prepared for NFS mounting
- Prometheus uses local volume with 30-day retention
- rsyslog logs stored in shared volume with Promtail

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