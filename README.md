# K3s-Sentinel: AI Agent for K3s Cluster Root Cause Analysis

K3s-Sentinel is an intelligent observability agent designed specifically for K3s clusters. It monitors cluster health, detects anomalies, and uses AI-powered analysis to trace symptoms back to their root causes.

## Quick Start

### 1. Clone or Navigate to Project

```bash
cd /workspace/k3s-sentinel
```

### 2. Configure Environment

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your settings
nano .env
```

**Required Settings:**
- `LLM_API_KEY` - Your LLM API key (OpenAI, Anthropic, etc.)

**Optional Settings:**
- `LLM_PROVIDER` - Provider: openai, anthropic, gemini, azure_openai, ollama
- `LLM_MODEL` - Model to use (default: gpt-4)
- Alert channels: Slack, Teams, Email, Webhooks, etc.

### 3. Start K3s-Sentinel

```bash
# Start all services (Backend API + Dashboard)
./start.sh

# Or start specific services:
./start.sh --backend   # Only backend API
./start.sh --frontend  # Only dashboard
```

### 4. Access Endpoints

After starting, access these endpoints:

| Service | URL | Description |
|---------|-----|-------------|
| **Dashboard** | http://localhost:3000 | Web UI for cluster monitoring |
| **Backend API** | http://localhost:8000 | REST API for cluster operations |
| **API Docs** | http://localhost:8000/docs | Interactive API documentation |

## Features

- **Real-time Monitoring**: Collects Kubernetes events, logs, and metrics
- **Dependency Graph**: Builds dynamic topology of cluster resources
- **AI-Powered Analysis**: Uses rule-based + LLM-powered root cause analysis
- **RAG Integration**: Leverages K3s troubleshooting knowledge base
- **Multi-channel Alerts**: Supports Slack, Teams, Email, Webhooks, and more
- **K3s-specific**: Built specifically for K3s components (Traefik, Local Path Provisioner, etc.)

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      K3s-Sentinel                            │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Telemetry   │  │   Context    │  │    Action     │     │
│  │  Collector   │──│   Engine     │──│   Dispatcher  │     │
│  │              │  │              │  │               │     │
│  │  - Events    │  │  - Topology  │  │  - Alerts     │     │
│  │  - Logs      │  │  - Vector DB │  │  - Webhooks   │     │
│  │  - Metrics   │  │  - History   │  │  - Reports    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│          │                 │                                 │
│          └────────┬────────┘                                 │
│                   ▼                                          │
│          ┌──────────────┐                                     │
│          │ Analysis Core│                                     │
│          │              │                                     │
│          │ - Symptom   │                                     │
│          │   Detection │                                     │
│          │ - RCA      │                                     │
│          │ - LLM      │                                     │
│          └──────────────┘                                     │
└─────────────────────────────────────────────────────────────┘
```

## API Endpoints

### Cluster Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Health check |
| GET | `/api/health` | Detailed health status |
| POST | `/api/cluster/validate` | Validate kubeconfig |
| POST | `/api/cluster/connect` | Connect to cluster |
| POST | `/api/cluster/disconnect` | Disconnect from cluster |
| GET | `/api/cluster/info` | Get cluster information |
| GET | `/api/cluster/nodes` | List all nodes |
| GET | `/api/cluster/pods` | List all pods |
| GET | `/api/cluster/services` | List all services |

### Alerts

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/alerts` | Get recent alerts with RCA |

### Example API Usage

```bash
# Check health
curl http://localhost:8000/api/health

# Get cluster info
curl http://localhost:8000/api/cluster/info

# Validate kubeconfig
curl -X POST http://localhost:8000/api/cluster/validate \
  -H "Content-Type: application/json" \
  -d '{"kubeconfig_path": "/path/to/kubeconfig"}'

# Get alerts
curl http://localhost:8000/api/alerts
```

## Configuration

All configuration is managed through the `.env` file. Copy `.env.example` to `.env` and customize:

### LLM Configuration

```env
LLM_PROVIDER=openai
LLM_API_KEY=your-api-key-here
LLM_MODEL=gpt-4
LLM_TEMPERATURE=0.7
LLM_MAX_TOKENS=2000
```

### Alert Configuration

```env
ALERTS_ENABLED=true
ALERT_MIN_SEVERITY=warning
ALERT_COOLDOWN_SECONDS=300

# Slack
SLACK_ENABLED=false
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...

# Teams
TEAMS_ENABLED=false
TEAMS_WEBHOOK_URL=https://outlook.office.com/webhook/...

# Email
EMAIL_ENABLED=false
EMAIL_SMTP_HOST=smtp.gmail.com
EMAIL_USERNAME=your-email@gmail.com
EMAIL_PASSWORD=your-password
```

### Agent Settings

```env
KUBECONFIG_PATH=~/.kube/config
POLL_INTERVAL=10
LOG_LEVEL=INFO
```

## Prerequisites

- **Python 3.9+**
- **Kubernetes/K3s cluster** (optional, for full functionality)
- **kubectl** configured (optional, for cluster operations)

## Installation

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Setup Virtual Environment (Optional)

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Supported Issue Types

K3s-Sentinel can detect and analyze:

- **Pod Issues**: CrashLoopBackOff, Evicted, Pending, ImagePullBackOff
- **Node Issues**: NotReady, DiskPressure, MemoryPressure
- **Network Issues**: Service unavailable, Ingress errors
- **Storage Issues**: PVC pending, mount failures
- **Configuration Issues**: ConfigMap/Secret missing, invalid specs

## Root Cause Analysis

The agent traces issues through the dependency graph:

```
Service → Deployment → Pod → PVC → Node
   │          │         │      │     │
   └──────────┴─────────┴──────┴─────┘
                   │
            Symptom (Effect)
                   │
                   ▼
            Root Cause
```

## Troubleshooting

### Agent Not Receiving Events

1. Check RBAC permissions:
```bash
kubectl auth can-i get events
```

2. Verify kubeconfig is correct:
```bash
kubectl config current-context
```

### LLM Analysis Not Working

1. Verify API key is set:
```bash
grep LLM_API_KEY .env
```

2. Check network connectivity to LLM provider

### Dashboard Not Loading

1. Check if backend is running:
```bash
curl http://localhost:8000/api/health
```
<img width="1908" height="1065" alt="image" src="https://github.com/user-attachments/assets/d006c021-fcaa-4ab0-be9f-e70676c161d9" />


2. Check dashboard logs:
```bash
tail -f logs/frontend.log
```

## License

MIT License

## Contributing

Contributions are welcome! Please read our contributing guidelines before submitting PRs.
