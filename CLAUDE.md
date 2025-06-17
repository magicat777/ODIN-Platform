# CLAUDE.md - ODIN Platform

This file provides guidance to Claude Code when working with the ODIN Platform monitoring system.

## Project Overview

ODIN (Omnipresent Diagnostics and Intelligence Network) is a comprehensive monitoring and observability platform built on Kubernetes (K3s). It provides metrics collection, log aggregation, visualization, and alerting capabilities that serve as the foundation for AI-powered automation.

## Technology Stack

- **Container Orchestration**: K3s (lightweight Kubernetes)
- **Metrics**: Prometheus + Node Exporter + cAdvisor
- **Logs**: Loki + Promtail
- **Visualization**: Grafana
- **Alerts**: Alertmanager
- **Scripting**: Python 3.10+ with virtual environment at /opt/odin/venv

## Project Structure

```
ODIN-Platform/
├── k8s/                    # Kubernetes manifests
│   ├── monitoring-*.yaml   # Monitoring stack configs
│   └── *-configmaps.yaml   # Service configurations
├── scripts/                # Utility scripts
│   ├── backup-*.sh        # Backup scripts
│   ├── storage-*.sh       # Storage management
│   └── setup-*.sh         # Setup utilities
├── docs/                   # Documentation
├── install.sh             # Installation script
└── REQUIREMENTS.md        # Detailed requirements
```

## Key Commands

### Development
```bash
# Check cluster status
kubectl get nodes
kubectl get pods -n monitoring

# View logs
kubectl logs -n monitoring <pod-name>

# Access services
# Prometheus: http://localhost:31493
# Grafana: http://localhost:31494 (admin/admin)
# Loki: http://localhost:31495
# Alertmanager: http://localhost:31496
```

### Storage Management
```bash
# Check PVC status
kubectl get pvc -n monitoring

# Backup data
./scripts/backup-k8s-pvcs.sh

# Monitor storage
./scripts/k8s-storage-status.sh
```

## Important Paths

- **Data Storage**: `/mnt/data/kubernetes/`
- **Project Files**: `/mnt/data/projects/ODIN/`
- **Backups**: `/mnt/data/backups/`
- **Python venv**: `/opt/odin/venv/`

## Alert Integration

ODIN sends alerts to Odin Prime via webhook:
- Alertmanager webhook URL: `http://odin-prime-alert-ingestion.odin-prime.svc.cluster.local/webhook`
- Alert format: Prometheus Alertmanager JSON
- Includes labels, annotations, and fingerprint

## Best Practices

1. **Storage**: Always use the 2TB secondary SSD (/mnt/data) for persistent data
2. **Monitoring**: Keep Prometheus retention at 30 days to manage storage
3. **Logs**: Loki retention at 7 days with proper label selectors
4. **Backups**: Run daily backups of Prometheus data and Grafana dashboards
5. **Updates**: Test updates in a separate namespace first

## Common Issues

1. **High Memory Usage**: Prometheus may need memory limits adjusted based on metric cardinality
2. **Storage Full**: Monitor PVC usage and expand if needed
3. **Slow Queries**: Add recording rules for frequently used queries
4. **Missing Metrics**: Check service discovery and scrape configs

## Testing

```bash
# Test Prometheus is scraping
curl http://localhost:31493/api/v1/targets

# Test Loki is receiving logs  
curl http://localhost:31495/loki/api/v1/query?query='{job="kube-system"}'

# Test Grafana datasources
curl -u admin:admin http://localhost:31494/api/datasources

# Test alert routing
curl -X POST http://localhost:31496/api/v1/alerts
```

## Security Notes

- Default Grafana password should be changed in production
- Consider enabling TLS for external access
- Use NetworkPolicies to restrict inter-service communication
- Regularly update all components for security patches

## Integration with Odin Prime

ODIN provides the observability layer that Odin Prime consumes:
- Prometheus alerts trigger Odin Prime's analysis pipeline
- Metrics API used for historical data during analysis
- Loki queries provide log context for incidents
- Grafana dashboards show Odin Prime's automation metrics

When working on ODIN Platform, remember it's the foundation that enables intelligent automation - reliability and performance are critical.