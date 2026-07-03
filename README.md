# Stack: Caddy + WordPress + Grafana + Prometheus + Watchtower + Portainer

## 1. DNS
Point these A records at your VPS IP:
- `mansatryobu.my.id`
- `www.mansatryobu.my.id`
- `grafana.mansatryobu.my.id`
- `prometheus.mansatryobu.my.id`
- `portainer.mansatryobu.my.id`
- `uptime.mansatryobu.my.id`

## 2. Firewall
Open ports 80 and 443 only (Caddy handles TLS via Let's Encrypt automatically). Do not expose Grafana/Prometheus/Portainer ports directly — only Caddy is public-facing.

## 3. Files
Copy `docker-compose.yml`, `Caddyfile`, `prometheus.yml`, and `.env.example` to a folder on the VPS, e.g. `/opt/stack/`.

```bash
mkdir -p /opt/stack && cd /opt/stack
# copy the 4 files here, then:
cp .env.example .env
nano .env   # set real passwords
```

## 4. Launch

```bash
docker compose up -d
```

## 5. First-time access
- WordPress: `https://mansatryobu.my.id` — run the install wizard
- Grafana: `https://grafana.mansatryobu.my.id` — login `admin` / your `GRAFANA_ADMIN_PASSWORD`
- Prometheus: `https://prometheus.mansatryobu.my.id`
- Portainer: `https://portainer.mansatryobu.my.id` — set admin password on first visit
- Uptime Kuma: `https://uptime.mansatryobu.my.id` — set admin account on first visit, then add monitors for your public URLs

## Monitoring stack
- **cAdvisor** and **node-exporter** are scraped by Prometheus automatically (see `prometheus.yml`) — container-level and host-level metrics, no extra config needed.
- In Grafana, add Prometheus as a data source (`http://prometheus:9090`) and import community dashboards: **cAdvisor dashboard ID 193** and **Node Exporter Full ID 1860** from grafana.com/dashboards.
- Uptime Kuma runs independently of Prometheus — add each public URL (WordPress, Grafana, etc.) as an HTTP monitor and configure a notification channel (Telegram/Discord/email) under Settings > Notifications.

## Notes
- Caddy auto-issues and renews TLS certs for each domain — no manual cert setup needed, just make sure DNS is live before starting.
- Watchtower checks for image updates once daily and restarts containers automatically (`WATCHTOWER_POLL_INTERVAL=86400`).
- Portainer, Grafana, and Uptime Kuma all listen on plain HTTP internally; Caddy terminates TLS so this is safe as long as those ports aren't published externally (they aren't in this compose file).
- cAdvisor needs `privileged: true` to read host cgroup/container stats — this is normal for cAdvisor and expected.
