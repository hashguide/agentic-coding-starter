---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, vps, system, setup, monitoring, agents]
authority: guidance
---

# VPS Monitoring Agents

System monitoring gives you visibility into the server's health before problems become outages. CPU, memory, disk, and network metrics should flow to a dashboard you can check — and ideally alert you when thresholds are exceeded.

## Option: Netdata (Recommended for Solo Builders)

Netdata is the lowest-effort monitoring installation available. It runs a daemon that collects hundreds of system metrics with zero configuration and provides a real-time web dashboard.

```bash
# One-line install
curl https://get.netdata.cloud/kickstart.sh > /tmp/netdata-kickstart.sh
sudo bash /tmp/netdata-kickstart.sh --nightly-channel

sudo systemctl enable netdata
sudo systemctl start netdata
```

Access the dashboard at `http://<server-ip>:19999` (temporarily allow port 19999 or use an SSH tunnel):

```bash
# Access via SSH tunnel (recommended — do not expose 19999 to the internet)
ssh -L 19999:localhost:19999 deploy@<server-ip>
# Then open http://localhost:19999 in your browser
```

**Key metrics to watch:**

| Metric | Warning threshold | Critical threshold |
|---|---|---|
| CPU utilization | >70% sustained | >90% sustained |
| Memory usage | >80% | >95% |
| Disk usage | >75% | >90% |
| Disk I/O wait | >20% | >40% |
| Network traffic | Depends on plan | At or near bandwidth cap |

## Option: Prometheus Node Exporter

For teams already running a Prometheus/Grafana stack:

```bash
# Download and install node_exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
sudo mv node_exporter-*/node_exporter /usr/local/bin/
sudo useradd -rs /bin/false node_exporter

# Create systemd service
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Node Exporter exposes metrics on port `9100`. Connect from a Prometheus instance scraping `<server-ip>:9100/metrics`.

## Log Shipping

Forward application logs to a centralized log service for searching and retention:

**Option: Grafana Loki + Promtail**

```bash
# Install Promtail (the Loki log shipper)
sudo apt install -y apt-transport-https software-properties-common
sudo apt install promtail -y

# Configure Promtail to ship Node.js app logs and Nginx access logs
sudo nano /etc/promtail/config.yml
```

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: https://logs-prod-us-central1.grafana.net/loki/api/v1/push
    basic_auth:
      username: <grafana-cloud-user-id>
      password: <grafana-cloud-api-key>

scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          __path__: /var/log/nginx/access.log

  - job_name: app
    static_configs:
      - targets: [localhost]
        labels:
          job: app
          __path__: /home/deploy/app/logs/*.log
```

**Option: Papertrail or Logtail** — simpler SaaS alternatives. Install the rsyslog daemon integration or use their hosted log shipper.

## Disk Space Alerts

Configure a cron-based disk space alert before a full disk causes an outage:

```bash
# Add to crontab: crontab -e
*/30 * * * * df -h / | awk 'NR==2{gsub(/%/,"",$5); if($5>80) system("echo \"Disk at \" $5 \"% on $(hostname)\" | mail -s \"[ALERT] Disk space\" ops@yourdomain.com")}'
```

Or rely on Netdata's built-in alert system, which handles this automatically.

## Related Documents

- [VPS auto-updates](auto-updates.md)
- [VPS monitoring](../monitoring.md)
- [Observability logging](../../../../observability/logging.md)
- [Observability uptime](../../../../observability/uptime.md)
