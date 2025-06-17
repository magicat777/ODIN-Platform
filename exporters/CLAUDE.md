# CLAUDE.md - Exporters Best Practices

This guide helps Claude Code work effectively with K3s exporters in the ODIN Platform.

## Quick Commands

### Check Exporter Health
```bash
# Use the pod manager script
~/k3s-pod-manager.sh health disk-io-exporter
~/k3s-pod-manager.sh health claude-code-exporter
~/k3s-pod-manager.sh health nvidia-gpu-exporter

# Check all exporters at once
for exporter in disk-io claude-code nvidia-gpu system-power temperature network-performance; do
    echo "=== $exporter-exporter ==="
    ~/k3s-pod-manager.sh health "$exporter-exporter"
    echo
done
```

### View Logs
```bash
# Last 50 lines (default)
~/k3s-pod-manager.sh logs disk-io-exporter

# Last 100 lines
~/k3s-pod-manager.sh logs claude-code-exporter 100
```

### Restart Exporter
```bash
~/k3s-pod-manager.sh restart disk-io-exporter
```

### Check Metrics
```bash
# Check if Prometheus is scraping
~/k3s-pod-manager.sh metrics disk-io-exporter

# Check actual metric values
curl -s "http://localhost:31493/api/v1/query?query=up{job='disk-io-exporter'}" | jq '.data.result[0].value[1]'
```

## Container Image Best Practices

All exporters should use base images with necessary tools:

```yaml
# Good: Python slim with utilities
image: python:3.11-slim
command: ["/bin/bash", "-c"]
args:
- |
  apt-get update && apt-get install -y --no-install-recommends procps wget
  pip install prometheus_client psutil
  python /scripts/exporter.py

# Bad: Minimal images without tools
image: python:3.11-alpine  # Missing many utilities
```

## Health Check Configuration

All exporters MUST include proper health checks:

```yaml
livenessProbe:
  httpGet:
    path: /metrics
    port: 9100  # Adjust per exporter
  initialDelaySeconds: 60
  periodSeconds: 30

readinessProbe:
  httpGet:
    path: /metrics
    port: 9100
  initialDelaySeconds: 30
  periodSeconds: 10
```

## Metric Types

Use correct Prometheus metric types:

```python
# Counter - for cumulative values (can only increase)
requests_total = Counter('http_requests_total', 'Total requests')

# Gauge - for values that can go up or down
temperature = Gauge('temperature_celsius', 'Current temperature')
memory_usage = Gauge('memory_usage_bytes', 'Current memory usage')

# Histogram - for distributions
request_duration = Histogram('request_duration_seconds', 'Request duration')
```

## Rate Calculations

**Important**: `rate()` only works with Counter metrics in Prometheus!

### Option 1: Use Counters (Preferred)
```python
from prometheus_client import Counter
bytes_total = Counter('disk_read_bytes_total', 'Total bytes read')
# Prometheus query: rate(disk_read_bytes_total[5m])
```

### Option 2: Pre-calculate rates in exporter
```python
# If using Gauges, calculate rate in the exporter
disk_read_rate = Gauge('disk_read_bytes_per_second', 'Read rate')

# In collection loop:
current_bytes = get_disk_bytes()
if previous_bytes is not None:
    rate = (current_bytes - previous_bytes) / time_elapsed
    disk_read_rate.set(rate)
previous_bytes = current_bytes
```

## Debugging Issues

### Exporter not starting
```bash
# Check logs
~/k3s-pod-manager.sh logs exporter-name

# Check pod events
kubectl describe pod -n monitoring -l app=exporter-name

# Manual exec for debugging
kubectl exec -it -n monitoring -l app=exporter-name -- /bin/bash
```

### Metrics not appearing
```bash
# Check if exporter is running
~/k3s-health-check.sh <pod-name> monitoring <process-name>

# Check Prometheus targets
curl -s http://localhost:31493/api/v1/targets | jq '.data.activeTargets[] | select(.labels.job=="exporter-name")'

# Check service discovery
kubectl get servicemonitor -n monitoring exporter-name -o yaml
```

### No data in Grafana
1. Check metric exists: `curl -s "http://localhost:31493/api/v1/query?query=metric_name"`
2. Check time range in Grafana
3. Verify query syntax (especially rate() with Counters only)
4. Check datasource variable: `${datasource}`

## Exporter Ports

Standard ports for ODIN exporters:
- 9100: Node Exporter (built-in)
- 9835: NVIDIA GPU Exporter
- 9836: System Power Exporter
- 9837: Temperature Exporter
- 9838: Network Performance Exporter
- 9839: Disk I/O Exporter
- 9840: Claude Code Exporter

## Safe Commands for Pod Inspection

Instead of using `pgrep` or other missing commands:

```bash
# Check if process is running
ps aux | grep -v grep | grep "process_name"

# Count processes
ps aux | grep -v grep | grep -c "process_name"

# Get process PID
ps aux | grep -v grep | grep "process_name" | awk '{print $2}'

# Check port listening
netstat -tlnp 2>/dev/null | grep :9100 || ss -tlnp | grep :9100

# Test metrics endpoint
wget -q -O - http://localhost:9100/metrics | head -10
```