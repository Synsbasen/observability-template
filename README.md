# Observability Template

A small, self-hosted observability stack using:
- Prometheus (metrics)
- Grafana (dashboards)
- Loki + Promtail (logs)
- Node Exporter (host metrics)

## What this template provides
- Docker Compose setup for all services
- Local-only binding for Prometheus and Loki by default
- Grafana datasource provisioning for Loki
- Example Prometheus scrape jobs for app exporters

## Prerequisites
- Docker and Docker Compose plugin installed
- Linux host (Promtail is configured to read Docker logs from host)

## Quick start
For a Linux server, this is the fastest path to get everything running:

1. Clone and enter the project:
```bash
git clone https://github.com/Synsbasen/observability-template.git observability-template
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

5. Open Grafana from your browser:
- Local server access: `http://localhost:4000`
- Remote server access: `http://<server-ip>:4000`

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

## Expose Grafana publicly with Nginx (Ubuntu)
Use Nginx as a reverse proxy in front of Grafana. For production, keep Grafana bound to localhost and only expose ports `80/443`.

1. Bind Grafana to localhost in `docker-compose.yml`:
```yaml
ports:
  - "127.0.0.1:4000:3000"
```

2. Install Nginx:
```bash
sudo apt update
sudo apt install -y nginx
```

3. Create Nginx site config at `/etc/nginx/sites-available/grafana`:
```nginx
server {
    listen 80;
    server_name grafana.example.com;

    location / {
        proxy_pass http://127.0.0.1:4000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

4. Enable the site and reload Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/grafana /etc/nginx/sites-enabled/grafana
sudo nginx -t
sudo systemctl reload nginx
```

5. Open firewall for web traffic (if UFW is enabled):
```bash
sudo ufw allow 'Nginx Full'
```

6. Add TLS with Let's Encrypt:
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d grafana.example.com
```

After this, Grafana is reachable at `https://grafana.example.com`.

## Security notes
- Do not commit `.env`.
- Do not commit real credentials in config files.
- Rotate any credential immediately if it is ever committed.
- See `SECURITY.md` for disclosure guidance.
