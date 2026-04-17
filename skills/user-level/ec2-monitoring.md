# EC2 Monitoring Stack — Prometheus, Loki, Grafana

Full observability setup for a Node.js app on Ubuntu EC2:
- **Metrics**: Node Exporter (host) + prom-client (app) → Prometheus → Grafana
- **Logs**: Promtail → Loki → Grafana

## Usage

```
/ec2-monitoring
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Ubuntu EC2 Instance                      │
│                                                                 │
│  ┌─────────────────┐    /api/metrics     ┌───────────────────┐  │
│  │  Node.js App    │ ──────────────────► │                   │  │
│  │  (prom-client)  │    port 3000        │   Prometheus      │  │
│  └─────────────────┘                    │   port 9090       │  │
│                                         │                   │  │
│  ┌─────────────────┐    /metrics        │   scrape_configs: │  │
│  │  Node Exporter  │ ──────────────────► │   - node_exporter │  │
│  │  (host metrics) │    port 9100        │   - nodejs app    │  │
│  └─────────────────┘                    └────────┬──────────┘  │
│                                                  │             │
│  ┌─────────────────┐                    ┌────────▼──────────┐  │
│  │  App Logs       │                    │                   │  │
│  │  /var/log/      │ ──► Promtail ─────► │      Loki         │  │
│  │  journald       │     port 9080       │   port 3100       │  │
│  └─────────────────┘                    └────────┬──────────┘  │
│                                                  │             │
└──────────────────────────────────────────────────│─────────────┘
                                                   │
                                          ┌────────▼──────────┐
                                          │     Grafana        │
                                          │     port 3000      │
                                          │                   │
                                          │  Data Sources:    │
                                          │  - Prometheus     │
                                          │  - Loki           │
                                          │                   │
                                          │  Dashboards:      │
                                          │  - Node Exporter  │
                                          │  - Node.js App    │
                                          │  - Logs Explorer  │
                                          └───────────────────┘
```

---

## Step 1: Node Exporter (host metrics)

```bash
# Download (check latest at github.com/prometheus/node_exporter/releases)
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

`/etc/systemd/system/node_exporter.service`:
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
# Verify: curl http://localhost:9100/metrics
```

---

## Step 2: prom-client in Node.js app

```bash
npm install prom-client
```

`src/lib/metrics.ts`:
```ts
import { Registry, collectDefaultMetrics } from 'prom-client';

const registry = new Registry();
registry.setDefaultLabels({ app: 'your-app-name' });
collectDefaultMetrics({ register: registry });

export { registry };
```

`src/instrumentation.ts`:
```ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./lib/metrics');
  }
}
```

`src/app/api/metrics/route.ts`:
```ts
import { NextResponse } from 'next/server';
import { registry } from '@/lib/metrics';

export const dynamic = 'force-dynamic';

export async function GET() {
  const metrics = await registry.metrics();
  return new NextResponse(metrics, {
    headers: { 'Content-Type': registry.contentType },
  });
}
```

```bash
# Verify: curl http://localhost:<app-port>/api/metrics
```

---

## Step 3: Prometheus

```bash
# Download (check latest at github.com/prometheus/prometheus/releases)
wget https://github.com/prometheus/prometheus/releases/download/v2.53.0/prometheus-2.53.0.linux-amd64.tar.gz
tar xvf prometheus-2.53.0.linux-amd64.tar.gz
sudo cp prometheus-2.53.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.53.0.linux-amd64/promtool /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo cp -r prometheus-2.53.0.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-2.53.0.linux-amd64/console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

`/etc/prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'nodejs-app'
    static_configs:
      - targets: ['localhost:<app-port>']
    metrics_path: '/api/metrics'
    scrape_interval: 15s
```

`/etc/systemd/system/prometheus.service`:
```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
# Verify: open http://<ec2-ip>:9090/targets — both jobs should show UP
```

---

## Step 4: Loki

```bash
# Download (check latest at github.com/grafana/loki/releases)
wget https://github.com/grafana/loki/releases/download/v3.1.0/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
sudo cp loki-linux-amd64 /usr/local/bin/loki
sudo useradd --no-create-home --shell /bin/false loki
sudo mkdir -p /etc/loki /var/lib/loki
sudo chown -R loki:loki /etc/loki /var/lib/loki
```

`/etc/loki/loki-config.yaml`:
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 5m
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2024-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /var/lib/loki/index
    cache_location: /var/lib/loki/cache
  filesystem:
    directory: /var/lib/loki/chunks

limits_config:
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
```

`/etc/systemd/system/loki.service`:
```ini
[Unit]
Description=Loki
After=network.target

[Service]
User=loki
Group=loki
Type=simple
ExecStart=/usr/local/bin/loki -config.file=/etc/loki/loki-config.yaml

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now loki
# Verify: curl http://localhost:3100/ready
```

---

## Step 5: Promtail

```bash
# Download same version as Loki
wget https://github.com/grafana/loki/releases/download/v3.1.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
sudo cp promtail-linux-amd64 /usr/local/bin/promtail
sudo useradd --no-create-home --shell /bin/false promtail
sudo mkdir -p /etc/promtail
sudo chown -R promtail:promtail /etc/promtail
# Allow promtail to read logs
sudo usermod -aG adm promtail
```

`/etc/promtail/promtail-config.yaml`:
```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log

  - job_name: journal
    journal:
      max_age: 12h
      labels:
        job: systemd-journal
    relabel_configs:
      - source_labels: [__journal__systemd_unit]
        target_label: unit

  # Add your app's log file if it writes to a file
  # - job_name: nodejs-app
  #   static_configs:
  #     - targets:
  #         - localhost
  #       labels:
  #         job: nodejs-app
  #         __path__: /var/log/your-app/*.log
```

`/etc/systemd/system/promtail.service`:
```ini
[Unit]
Description=Promtail
After=network.target

[Service]
User=promtail
Group=promtail
Type=simple
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail/promtail-config.yaml

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now promtail
```

---

## Step 6: Grafana

```bash
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://apt.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install -y grafana
sudo systemctl enable --now grafana-server
# Default login: admin / admin
```

### Add data sources (in Grafana UI)

1. **Connections → Data Sources → Add → Prometheus**
   - URL: `http://localhost:9090`
   - Save & Test

2. **Connections → Data Sources → Add → Loki**
   - URL: `http://localhost:3100`
   - Save & Test

### Import dashboards

| Dashboard | ID | What it shows |
|-----------|-----|---------------|
| Node Exporter Full | `1860` | CPU, RAM, disk, network — host level |
| Node.js Application | `11159` | Heap, event loop, GC — app level |
| Loki Logs | Use Explore tab | Query logs with LogQL |

---

## EC2 Security Group — Required Inbound Ports

| Port | Service | Source |
|------|---------|--------|
| 3000 | Grafana | Your IP |
| 9090 | Prometheus | Your IP (or localhost only) |
| 3100 | Loki | localhost only |
| 9100 | Node Exporter | localhost only |
| 9080 | Promtail | localhost only |

> Loki, Node Exporter, and Promtail should **not** be exposed publicly — only Grafana and optionally Prometheus need inbound rules.

---

## Verify everything is running

```bash
sudo systemctl status node_exporter prometheus loki promtail grafana-server
```

All five should show `active (running)`.
