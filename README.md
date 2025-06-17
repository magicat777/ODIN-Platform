# ODIN - Omnipresent Diagnostics and Intelligence Network

[![Status](https://img.shields.io/badge/Status-Production%20Ready-success)](https://github.com/odin-prime)
[![Platform](https://img.shields.io/badge/Platform-Ubuntu%2022.04-orange)](https://ubuntu.com)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-K3s%20v1.32.5-blue)](https://k3s.io)
[![GPU](https://img.shields.io/badge/GPU-NVIDIA%20RTX%204080-green)](https://nvidia.com)
[![AI](https://img.shields.io/badge/AI-Claude%20%2B%20Local%20LLMs-purple)](https://anthropic.com)

ODIN is a comprehensive monitoring and automation platform that combines traditional observability tools with cutting-edge AI capabilities to create a self-healing infrastructure. Running on a dual-boot Razer Blade 18 with dual SSDs, it leverages Kubernetes, NVIDIA GPUs, and multiple AI models to automatically detect, analyze, and remediate system issues.

## 🚀 Key Features

### 📊 Comprehensive Monitoring
- **Metrics**: Prometheus with 25+ custom alert rules
- **Logs**: Loki with real-time aggregation and search
- **Visualization**: 15+ Grafana dashboards
- **Alerts**: Multi-channel notifications (email, desktop, webhooks)

### 🤖 AI-Powered Automation
- **Multi-Tier Analysis**: Local LLMs (Phi-3.5) → Claude AI escalation
- **Pattern Matching**: ChromaDB vector similarity (<10ms response)
- **Automated Remediation**: Safe actions with human approval for risky operations
- **Learning System**: Playbook library that learns from successful automations

### 🛡️ Safety & Compliance
- **Risk Assessment**: 3-tier classification (safe/moderate/dangerous)
- **Approval Workflow**: Web UI for human oversight
- **Rollback Protection**: Automatic snapshots before changes
- **Audit Trail**: Complete logging with compliance reporting

### 🎮 GPU Acceleration
- **NVIDIA RTX 4080**: 16GB VRAM for ML inference
- **CUDA Integration**: GPU-accelerated anomaly detection
- **Power Monitoring**: Real-time GPU metrics and thermal management

## 🏗️ System Architecture

### Hardware Configuration
- **Platform**: Razer Blade 18 (RZ09-0484)
- **CPU**: Intel Core i9 (16 cores)
- **RAM**: 32GB DDR5
- **GPU**: NVIDIA RTX 4080 (16GB VRAM)
- **Storage**: 
  - Primary SSD: 360GB (Ubuntu 22.04 + Windows 11 dual-boot)
  - Secondary SSD: 2TB (All ODIN data and Kubernetes volumes)
- **Network Storage**: Synology NAS (3.3TB for archives)

### Storage Layout
```
/mnt/data/ (2TB SSD)
├── projects/ODIN/      # Main project (symlinked to ~/projects/ODIN)
├── kubernetes/         # Persistent volumes (260Gi allocated)
│   ├── prometheus/     # 100Gi - Metrics storage
│   ├── loki/          # 50Gi - Log storage
│   ├── grafana/       # 20Gi - Dashboards
│   └── odin-prime/    # 90Gi - AI/ML data
├── backups/           # Automated backups
└── docker/            # Container volumes
```

## 📋 Prerequisites

- Ubuntu 22.04 LTS
- NVIDIA GPU with CUDA 12.7+ support
- 32GB+ RAM recommended
- 2TB+ storage for data
- Kubernetes cluster (K3s included in setup)

## 🚀 Quick Start

### 1. Clone Repository
```bash
cd /mnt/data/projects
git clone https://github.com/yourusername/ODIN.git
ln -s /mnt/data/projects/ODIN ~/projects/ODIN
cd ~/projects/ODIN
```

### 2. Run Setup Script
```bash
./scripts/quickstart.sh
```

This will:
- Install K3s with NVIDIA runtime
- Deploy monitoring stack
- Configure storage on 2TB drive
- Set up ODIN Prime automation

### 3. Access Services

#### Core Monitoring
- Grafana: http://localhost:31494 (admin/admin)
- Prometheus: http://localhost:31493
- AlertManager: http://localhost:31495

#### ODIN Prime Automation
```bash
# Port forward to access UIs
kubectl port-forward -n odin-prime svc/approval-api 30915:8090
kubectl port-forward -n odin-prime svc/audit-viewer 30916:8096
```

Then visit:
- Approval UI: http://localhost:30915
- Audit Viewer: http://localhost:30916

## 🏛️ Architecture Components

### Monitoring Stack
| Component | Purpose | Resources | Storage |
|-----------|---------|-----------|---------|
| Prometheus | Metrics collection | 2Gi RAM, 500m CPU | 100Gi PV |
| Grafana | Visualization | 512Mi RAM, 250m CPU | 20Gi PV |
| Loki | Log aggregation | 1Gi RAM, 500m CPU | 50Gi PV |
| AlertManager | Alert routing | 256Mi RAM, 100m CPU | ConfigMap |

### AI/ML Stack
| Component | Purpose | Model | Resources |
|-----------|---------|-------|-----------|
| Pattern Matcher | Vector similarity | ChromaDB | 1Gi RAM, 10Gi PV |
| Tier 1 Triage | Quick classification | Phi-3.5 | 2Gi RAM, GPU |
| Claude Service | Complex analysis | Claude API | Rate limited |
| GPU Inference | Model hosting | Various | 8Gi RAM, GPU |

### Anomaly Detectors
- **GPU Anomaly**: Temperature and power analysis
- **Pod Anomaly**: Container lifecycle monitoring
- **Process Anomaly**: Security behavior detection
- **Disk Anomaly**: Space forecasting and I/O patterns

## 📁 Project Structure

```
ODIN/
├── k8s/                 # Kubernetes manifests
│   ├── monitoring/      # Prometheus, Grafana, Loki
│   ├── odin-prime/      # AI automation components
│   └── storage/         # PV/PVC definitions
├── odin-prime/          # AI automation system
│   ├── src/            # Python source code
│   ├── config/         # Configuration files
│   └── scripts/        # Deployment scripts
├── scripts/            # Utility scripts
│   ├── storage-management.sh
│   ├── k8s-storage-status.sh
│   └── quickstart.sh
├── docs/               # Documentation
│   ├── RAZER_BLADE_18_SYSTEM_OVERVIEW.md
│   ├── K8S_STORAGE_MIGRATION_GUIDE.md
│   └── EMAIL_ALERTS_IMPLEMENTATION.md
└── backups/            # Local backup storage
```

## 🔧 Storage Management

### Check Storage Status
```bash
./scripts/storage-management.sh status
```

### View Kubernetes Storage
```bash
./scripts/k8s-storage-status.sh
```

### Create Backup
```bash
./scripts/storage-management.sh backup
```

### Clean Old Data
```bash
./scripts/storage-management.sh clean
```

## 🤖 AI Automation Workflow

1. **Alert Detection**: Prometheus/Loki → AlertManager
2. **Initial Analysis**: Pattern Matcher checks known issues
3. **AI Triage**: Local LLMs classify and analyze
4. **Complex Cases**: Escalate to Claude API
5. **Risk Assessment**: Classify proposed actions
6. **Execution**: Apply changes with safety checks
7. **Learning**: Save successful patterns as playbooks

## 🛡️ Safety Features

- **Approval Required**: Moderate-risk actions need human approval
- **Automatic Rollback**: Failed changes revert automatically
- **Resource Limits**: Hard caps on scaling operations
- **Audit Trail**: Every action logged with context
- **Cost Controls**: $50/day Claude API budget

## 📊 Monitoring Dashboards

### System Overview
- CPU, Memory, Disk usage
- Network traffic
- Container metrics
- GPU utilization

### ODIN Prime Analytics
- Alert processing pipeline
- Automation success rates
- Cost tracking
- Pattern match statistics

### Security Monitoring
- Process anomalies
- Failed automations
- Approval requests
- Audit compliance

## 🔐 Security Considerations

- All services bind to localhost only
- K8s RBAC enabled
- Secrets managed via Kubernetes
- Automatic redaction of sensitive data
- Full audit trail for compliance

## 📚 Documentation

- [System Overview](docs/RAZER_BLADE_18_SYSTEM_OVERVIEW.md) - Complete hardware/software architecture
- [Storage Migration](docs/K8S_STORAGE_MIGRATION_GUIDE.md) - Moving to 2TB SSD
- [Email Alerts](docs/EMAIL_ALERTS_IMPLEMENTATION.md) - Setting up notifications
- [CLAUDE.md](CLAUDE.md) - AI assistant guidance

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📈 Performance Metrics

- **Alert Response**: <200ms for pattern matching
- **AI Analysis**: <10s including Claude API
- **Automation Execution**: <60s typical
- **Storage**: 1.7TB available on dedicated SSD
- **Uptime**: 99.9% for core services

## 🔍 Troubleshooting

### Check Service Health
```bash
kubectl get pods -n monitoring
kubectl get pods -n odin-prime
```

### View Logs
```bash
kubectl logs -n odin-prime deployment/alert-ingestion
kubectl logs -n monitoring deployment/prometheus
```

### Storage Issues
```bash
df -h /mnt/data
./scripts/storage-management.sh status
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Anthropic for Claude AI integration
- NVIDIA for GPU support
- K3s for lightweight Kubernetes
- The open-source monitoring community

---

**ODIN Prime**: Where traditional monitoring meets AI-powered automation, running on optimized dual-SSD storage for maximum performance and reliability.
