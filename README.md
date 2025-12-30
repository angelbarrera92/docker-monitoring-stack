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

### Prometheus

| Variable | Default | Description |
|----------|---------|-------------|
| `PROMETHEUS_VERSION` | `v2.48.0` | Prometheus Docker image version |
| `PROMETHEUS_CONTAINER_NAME` | `prometheus` | Container name |
| `PROMETHEUS_PORT` | `9090` | Host port for Prometheus UI |
| `PROMETHEUS_RETENTION` | `15d` | Data retention period |
| `PROMETHEUS_DATA_PATH` | `prometheus_data` | Named volume or host path (e.g., `/data/prometheus`) |
| `PROMETHEUS_NETWORK_MODE` | *(empty)* | Set to `host` for VPN access |

### Grafana

| Variable | Default | Description |
|----------|---------|-------------|
| `GRAFANA_VERSION` | `10.2.3` | Grafana Docker image version |
| `GRAFANA_CONTAINER_NAME` | `grafana` | Container name |
| `GRAFANA_PORT` | `3000` | Host port for Grafana UI |
| `GRAFANA_DATA_PATH` | `grafana_data` | Named volume or host path |
| `PROMETHEUS_URL` | `http://prometheus:9090` | Prometheus URL for datasource (see note below) |
| `GRAFANA_ROOT_URL` | `http://localhost:3000` | Public URL (important for OAuth) |
| `GRAFANA_DOMAIN` | `localhost` | Domain name |
| `GRAFANA_ADMIN_USER` | `admin` | Admin username |
| `GRAFANA_ADMIN_PASSWORD` | `admin` | Admin password (change in production!) |
| `GRAFANA_ALLOW_SIGN_UP` | `false` | Allow user registration |
| `GF_SECURITY_DISABLE_INITIAL_ADMIN_CREATION` | `false` | Disable initial admin creation (OAuth-only) |
| `GF_AUTH_BASIC_ENABLED` | `true` | Enable/disable basic auth (local login) |
| `GF_AUTH_DISABLE_LOGIN_FORM` | `false` | Disable login form (OAuth-only) |
| `GF_INSTALL_PLUGINS` | *(empty)* | Comma-separated list of plugins to install |

> **Note:** When Prometheus runs in host network mode (`PROMETHEUS_NETWORK_MODE=host`), set `PROMETHEUS_URL` to reach it from Grafana:
> - Docker Desktop: `http://host.docker.internal:9090`
> - Linux: `http://172.17.0.1:9090` or your host's actual IP

### GitHub OAuth

| Variable | Default | Description |
|----------|---------|-------------|
| `GF_AUTH_GITHUB_ENABLED` | `false` | Enable GitHub OAuth |
| `GF_AUTH_GITHUB_CLIENT_ID` | *(empty)* | GitHub OAuth App Client ID |
| `GF_AUTH_GITHUB_CLIENT_SECRET` | *(empty)* | GitHub OAuth App Client Secret |
| `GF_AUTH_GITHUB_ALLOWED_ORGANIZATIONS` | *(empty)* | Restrict to GitHub org members (e.g., `myorg`) |
| `GF_AUTH_GITHUB_ALLOW_SIGN_UP` | `true` | Allow new users via GitHub |
| `GF_AUTH_GITHUB_AUTO_LOGIN` | `false` | Skip login page, go directly to GitHub |
| `GF_AUTH_GITHUB_ROLE_ATTRIBUTE_PATH` | *(empty)* | JMESPath expression for role mapping |
| `GF_AUTH_GITHUB_ALLOW_ASSIGN_GRAFANA_ADMIN` | `false` | Allow GitHub org admins to be Grafana admins |
| `GF_AUTH_GITHUB_ROLE_ATTRIBUTE_STRICT` | `false` | Enforce strict role mapping |

### General

| Variable | Default | Description |
|----------|---------|-------------|
| `RESTART_POLICY` | `unless-stopped` | Container restart policy |
| `MONITORING_NETWORK` | `monitoring` | Docker network name |

## GitHub OAuth Setup

1. **Create a GitHub OAuth App:**
   - Go to [GitHub Developer Settings](https://github.com/settings/developers)
   - Click "New OAuth App"
   - Set **Homepage URL**: `https://grafana.yourdomain.com`
   - Set **Authorization callback URL**: `https://grafana.yourdomain.com/login/github`

2. **Configure your `.env`:**
   ```bash
   GF_AUTH_GITHUB_ENABLED=true
   GF_AUTH_GITHUB_CLIENT_ID=your_client_id
   GF_AUTH_GITHUB_CLIENT_SECRET=your_client_secret
   GF_AUTH_GITHUB_ALLOWED_ORGANIZATIONS=your-github-org
   GRAFANA_ROOT_URL=https://grafana.yourdomain.com
   GRAFANA_DOMAIN=grafana.yourdomain.com
   ```

3. **Grant access** by adding users to your GitHub organization.

## Reverse Proxy Setup

When exposing Grafana publicly (required for OAuth), you need a reverse proxy with HTTPS. See `docker-compose.override.yml.example` for complete Traefik and Caddy examples.

### Quick Caddy Setup

1. Create `docker-compose.override.yml`:
   ```yaml
   services:
     caddy:
       image: caddy:2-alpine
       container_name: caddy
       restart: unless-stopped
       ports:
         - "80:80"
         - "443:443"
       volumes:
         - ./caddy/Caddyfile:/etc/caddy/Caddyfile:ro
         - caddy_data:/data

     grafana:
       ports: !reset []

   volumes:
     caddy_data:
   ```

2. Create `caddy/Caddyfile`:
   ```
   grafana.yourdomain.com {
       reverse_proxy grafana:3000
   }
   ```

3. Update `.env`:
   ```bash
   GRAFANA_ROOT_URL=https://grafana.yourdomain.com
   GRAFANA_DOMAIN=grafana.yourdomain.com
   ```

Caddy automatically handles HTTPS certificates via Let's Encrypt.

### VPN / External Network Access

To scrape targets over VPN tunnels, you have two options:

**Option 1: Host network mode (recommended)**

Set `PROMETHEUS_NETWORK_MODE=host` in your `.env` file. This gives Prometheus direct access to host network interfaces (`tailscale0`, `wg0`, `tun0`, etc.).

**Option 2: Docker Compose override**

Copy `docker-compose.override.yml.example` to `docker-compose.override.yml` and customize for advanced networking scenarios.

## Services

- **Prometheus**: Metrics collection and storage (port 9090)
- **Grafana**: Visualization and dashboards (port 3000)

## Access

- Prometheus UI: `http://localhost:9090`
- Grafana UI: `http://localhost:3000` (default: admin/admin)
