# Observability Template

A small, self-hosted observability stack using:
- Prometheus (metrics)
- Grafana (dashboards)
- Loki + Promtail (logs)
- Node Exporter (host metrics)
- Cloudflare Tunnel (public access to Grafana over HTTPS without exposing ports)

## What this template provides
- Docker Compose setup for all services
- No public Docker port exposure by default
- Grafana published securely through Cloudflare Tunnel
- Grafana datasource provisioning for Loki
- Example Prometheus scrape jobs for app exporters

## Prerequisites
- Docker and Docker Compose plugin installed
- Linux host (Promtail is configured to read Docker logs from host)
- A Cloudflare account
- A domain managed in Cloudflare
- A Cloudflare Tunnel created in Cloudflare Zero Trust

## Quick start
For a Linux server, this is the fastest path to get everything running:

1. Clone and enter the project:
```bash
git clone <your-repo-url> observability-template
cd observability-template
```

2. Create your environment file:
```bash
cp .env.example .env
```

3. Start the stack:
```bash
docker compose up -d
```

4. Confirm services are healthy:
```bash
docker compose ps
```

5. Add a route to your published application in Cloudflare Tunnels:
Service URL should be `http://grafana:3000`.

## Stop
```bash
docker compose down
```

## Required/optional environment variables
Grafana SMTP configuration is env-based so credentials are not committed:
- `GF_SMTP_ENABLED` (default: `false`)
- `GF_SMTP_HOST` (example: `smtp.example.com:587`)
- `GF_SMTP_USER` (example: `apikey`)
- `GF_SMTP_PASSWORD`
- `GF_SMTP_FROM_ADDRESS`
- `GF_SMTP_FROM_NAME`
- `GF_SMTP_SKIP_VERIFY` (default: `false`)

## Prometheus app integration
Applications exposing Prometheus metrics should be reachable from this Docker network:
- `observability-network`

Example app scrape targets are defined in `config/prometheus.yml` and should be replaced with your own service names.
