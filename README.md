# Docker Monitoring Stack

A simple, configurable Docker monitoring stack that can be deployed on remote VMs with environment variable overrides.

## Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd docker-monitoring-stack

# Copy and customize environment variables (optional)
cp .env.example .env

# Start the stack
docker compose up -d
```

## Configuration

All configuration can be overridden using environment variables. Copy `.env.example` to `.env` and modify as needed.

| Variable | Default | Description |
|----------|---------|-------------|
| `PROMETHEUS_VERSION` | `v2.48.0` | Prometheus Docker image version |
| `PROMETHEUS_CONTAINER_NAME` | `prometheus` | Container name |
| `PROMETHEUS_PORT` | `9090` | Host port for Prometheus UI |
| `PROMETHEUS_RETENTION` | `15d` | Data retention period |
| `PROMETHEUS_DATA_PATH` | `prometheus_data` | Named volume or host path (e.g., `/data/prometheus`) |
| `RESTART_POLICY` | `unless-stopped` | Container restart policy |
| `MONITORING_NETWORK` | `monitoring` | Docker network name |
| `PROMETHEUS_NETWORK_MODE` | *(empty)* | Set to `host` for VPN access (Tailscale/WireGuard/OpenVPN) |

### VPN / External Network Access

To scrape targets over VPN tunnels, you have two options:

**Option 1: Host network mode (recommended)**

Set `PROMETHEUS_NETWORK_MODE=host` in your `.env` file. This gives Prometheus direct access to host network interfaces (`tailscale0`, `wg0`, `tun0`, etc.).

**Option 2: Docker Compose override**

Copy `docker-compose.override.yml.example` to `docker-compose.override.yml` and customize for advanced networking scenarios.

## Services

- **Prometheus**: Metrics collection and storage (port 9090)

## Access

- Prometheus UI: `http://localhost:9090`
