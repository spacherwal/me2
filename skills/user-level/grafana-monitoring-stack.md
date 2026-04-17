---
name: prometheus-grafana-monitoring-stack
description: >
  Step-by-step guide to install and configure a full monitoring stack
  (Prometheus + Grafana + Loki + Promtail + Node Exporter) on an Ubuntu
  server. Use this skill when you need to set up server metrics monitoring,
  log viewing, dashboards, or alerts for any application running on Ubuntu.
  Covers installation, configuration, port planning, AWS Security Group setup,
  and Grafana dashboard import. Works on same server as existing apps if RAM
  is above 2GB.
---


================================================================================
MONITORING STACK SETUP GUIDE
Prometheus + Grafana + Loki + Promtail + Node Exporter on Ubuntu
================================================================================

Written for: QA / DevOps engineers setting up monitoring for the first time
Server type: Ubuntu (AWS EC2)
All tools are FREE and open source


--------------------------------------------------------------------------------
WHAT EACH TOOL DOES
--------------------------------------------------------------------------------

Tool            Role
-----------     ----------------------------------------------------------------
Node Exporter   Collects server metrics (CPU, RAM, Disk) and exposes them
Prometheus      Scrapes and stores metrics from Node Exporter and your app
Loki            Stores logs (like Prometheus but for logs, not metrics)
Promtail        Ships logs from your server (PM2, Nginx) into Loki
Grafana         Dashboard UI that visualizes data from Prometheus and Loki

Flow:
    Your Server
        Node Exporter  --metrics-->  Prometheus  --\
        Promtail       --logs-->     Loki         ----> Grafana Dashboard
        Next.js app    --metrics-->  Prometheus  --/


--------------------------------------------------------------------------------
PORT PLAN (memorize this to avoid conflicts)
--------------------------------------------------------------------------------

Port    Service             Notes
-----   -----------------   ---------------------------------------------------
8080    Next.js app         Already running (do not change)
80      Nginx               Already running (do not change)
443     Nginx HTTPS         Already running (do not change)
3000    Grafana             Dashboard UI (open this in browser)
9090    Prometheus          Metrics storage (open this in browser)
9100    Node Exporter       Server metrics (internal only)
3100    Loki                Log storage (internal only)
9080    Promtail            Log shipper (internal only)

IMPORTANT: Only open ports 3000 and 9090 in AWS Security Group.
           Never expose 9100, 3100, 9080 to the public internet.


--------------------------------------------------------------------------------
PRE-CHECK: VERIFY SERVER HAS ENOUGH RESOURCES
--------------------------------------------------------------------------------

Run these before starting:

    free -h
    df -h

Minimum required:
    RAM:  2GB free  (check "available" column in free -h output)
    Disk: 5GB free  (check "Avail" column in df -h output)

If resources are insufficient, use Grafana Cloud (free tier) instead.
See ALTERNATIVE section at the bottom of this file.


--------------------------------------------------------------------------------
PHASE 1 - CREATE MONITORING DIRECTORY
--------------------------------------------------------------------------------

    mkdir -p /opt/monitoring
    cd /opt/monitoring


--------------------------------------------------------------------------------
PHASE 2 - INSTALL NODE EXPORTER (Server Metrics)
--------------------------------------------------------------------------------

STEP 1 - Download and extract:

    cd /opt/monitoring
    wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
    tar xvf node_exporter-1.7.0.linux-amd64.tar.gz

STEP 2 - Verify binary exists:

    ls /opt/monitoring/node_exporter-1.7.0.linux-amd64/node_exporter

STEP 3 - Create systemd service:

    nano /etc/systemd/system/node_exporter.service

    Paste this content:
    --------------------
    [Unit]
    Description=Node Exporter
    After=network.target

    [Service]
    User=root
    ExecStart=/opt/monitoring/node_exporter-1.7.0.linux-amd64/node_exporter

    [Install]
    WantedBy=multi-user.target
    --------------------

STEP 4 - Start and enable:

    sudo systemctl daemon-reload
    sudo systemctl enable node_exporter
    sudo systemctl start node_exporter

STEP 5 - Verify (should return metric lines):

    curl localhost:9100/metrics | head -5

TROUBLESHOOT - If curl fails:

    sudo systemctl status node_exporter
    journalctl -u node_exporter -n 50

    Common fix - wrong path. Check binary location:
        ls /opt/monitoring/node_exporter-1.7.0.linux-amd64/node_exporter
    Then update ExecStart in the service file to match exact path.


--------------------------------------------------------------------------------
PHASE 3 - INSTALL PROMETHEUS (Metrics Storage)
--------------------------------------------------------------------------------

STEP 1 - Download and extract:

    cd /opt/monitoring
    wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
    tar xvf prometheus-2.45.0.linux-amd64.tar.gz

STEP 2 - Create config file:

    nano /opt/monitoring/prometheus-2.45.0.linux-amd64/prometheus.yml

    Paste this content:
    --------------------
    global:
      scrape_interval: 15s

    scrape_configs:
      - job_name: 'server'
        static_configs:
          - targets: ['localhost:9100']

      - job_name: 'nextjs'
        static_configs:
          - targets: ['localhost:8080']
    --------------------

    NOTE: Change port 8080 if your app runs on a different port.

STEP 3 - Create systemd service:

    nano /etc/systemd/system/prometheus.service

    Paste this content:
    --------------------
    [Unit]
    Description=Prometheus
    After=network.target

    [Service]
    User=root
    ExecStart=/opt/monitoring/prometheus-2.45.0.linux-amd64/prometheus \
      --config.file=/opt/monitoring/prometheus-2.45.0.linux-amd64/prometheus.yml \
      --storage.tsdb.path=/opt/monitoring/prometheus-data \
      --storage.tsdb.retention.time=15d

    [Install]
    WantedBy=multi-user.target
    --------------------

STEP 4 - Start and enable:

    mkdir -p /opt/monitoring/prometheus-data
    sudo systemctl daemon-reload
    sudo systemctl enable prometheus
    sudo systemctl start prometheus

STEP 5 - Verify:

    curl localhost:9090
    (should return HTML page)


--------------------------------------------------------------------------------
PHASE 4 - INSTALL LOKI (Log Storage)
--------------------------------------------------------------------------------

STEP 1 - Download:

    cd /opt/monitoring
    wget https://github.com/grafana/loki/releases/download/v2.9.0/loki-linux-amd64.zip
    sudo apt install unzip -y
    unzip loki-linux-amd64.zip

STEP 2 - Create data directories:

    mkdir -p /opt/monitoring/loki-data/chunks
    mkdir -p /opt/monitoring/loki-data/rules
    mkdir -p /opt/monitoring/loki-data/index

STEP 3 - Create config file:

    nano /opt/monitoring/loki-config.yml

    Paste this content:
    --------------------
    auth_enabled: false

    server:
      http_listen_port: 3100
      grpc_listen_port: 9096

    common:
      path_prefix: /opt/monitoring/loki-data
      storage:
        filesystem:
          chunks_directory: /opt/monitoring/loki-data/chunks
          rules_directory: /opt/monitoring/loki-data/rules
      replication_factor: 1
      ring:
        instance_addr: 127.0.0.1
        kvstore:
          store: inmemory

    schema_config:
      configs:
        - from: 2020-10-24
          store: boltdb-shipper
          object_store: filesystem
          schema: v11
          index:
            prefix: index_
            period: 24h

    ruler:
      alertmanager_url: http://localhost:9093
    --------------------

STEP 4 - Create systemd service:

    nano /etc/systemd/system/loki.service

    Paste this content:
    --------------------
    [Unit]
    Description=Loki
    After=network.target

    [Service]
    User=root
    ExecStart=/opt/monitoring/loki-linux-amd64 -config.file=/opt/monitoring/loki-config.yml

    [Install]
    WantedBy=multi-user.target
    --------------------

STEP 5 - Start and enable:

    sudo systemctl daemon-reload
    sudo systemctl enable loki
    sudo systemctl start loki

STEP 6 - Verify:

    curl localhost:3100/ready
    (should return: ready)

TROUBLESHOOT - If Loki fails with mkdir error:

    rm -rf /opt/monitoring/loki-data
    mkdir -p /opt/monitoring/loki-data/chunks
    mkdir -p /opt/monitoring/loki-data/rules
    mkdir -p /opt/monitoring/loki-data/index
    sudo systemctl restart loki
    sudo systemctl status loki


--------------------------------------------------------------------------------
PHASE 5 - INSTALL PROMTAIL (Log Shipper)
--------------------------------------------------------------------------------

STEP 1 - Download:

    cd /opt/monitoring
    wget https://github.com/grafana/loki/releases/download/v2.9.0/promtail-linux-amd64.zip
    unzip promtail-linux-amd64.zip

STEP 2 - Create config file:

    nano /opt/monitoring/promtail-config.yml

    Paste this content:
    --------------------
    server:
      http_listen_port: 9080

    positions:
      filename: /opt/monitoring/positions.yaml

    clients:
      - url: http://localhost:3100/loki/api/v1/push

    scrape_configs:
      - job_name: nextjs
        static_configs:
          - targets:
              - localhost
            labels:
              job: nextjs
              app: your-app-name
              __path__: /root/.pm2/logs/*.log

      - job_name: nginx_access
        static_configs:
          - targets:
              - localhost
            labels:
              job: nginx
              type: access
              __path__: /var/log/nginx/access.log

      - job_name: nginx_errors
        static_configs:
          - targets:
              - localhost
            labels:
              job: nginx
              type: error
              __path__: /var/log/nginx/error.log
    --------------------

    NOTE: Change "your-app-name" to your actual PM2 app name.
          Change PM2 logs path if your user is not root:
          root user:    /root/.pm2/logs/*.log
          other user:   /home/username/.pm2/logs/*.log

STEP 3 - Create systemd service:

    nano /etc/systemd/system/promtail.service

    Paste this content:
    --------------------
    [Unit]
    Description=Promtail
    After=network.target

    [Service]
    User=root
    ExecStart=/opt/monitoring/promtail-linux-amd64 -config.file=/opt/monitoring/promtail-config.yml

    [Install]
    WantedBy=multi-user.target
    --------------------

STEP 4 - Start and enable:

    sudo systemctl daemon-reload
    sudo systemctl enable promtail
    sudo systemctl start promtail


--------------------------------------------------------------------------------
PHASE 6 - INSTALL GRAFANA (Dashboard UI)
--------------------------------------------------------------------------------

STEP 1 - Install:

    sudo apt install -y apt-transport-https software-properties-common
    wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
    echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
    sudo apt update
    sudo apt install grafana -y

STEP 2 - Start and enable:

    sudo systemctl enable grafana-server
    sudo systemctl start grafana-server

STEP 3 - Verify:

    sudo systemctl status grafana-server
    curl localhost:3000
    (should return HTML)


--------------------------------------------------------------------------------
PHASE 7 - VERIFY ALL SERVICES
--------------------------------------------------------------------------------

Run this to check all at once:

    sudo systemctl status node_exporter prometheus loki promtail grafana-server

All should show:  Active: active (running)

Quick port check:

    curl localhost:9100/metrics | head -3    # Node Exporter
    curl localhost:9090                      # Prometheus
    curl localhost:3100/ready                # Loki
    curl localhost:3000                      # Grafana


--------------------------------------------------------------------------------
PHASE 8 - OPEN PORTS IN AWS SECURITY GROUP
--------------------------------------------------------------------------------

Only open ports 3000 (Grafana) and 9090 (Prometheus) — others stay internal.

STEP 1 - Find your EC2 instance:
    AWS Console -> EC2 -> Instances
    Find your server by IP or name
    Click on Instance ID

STEP 2 - Go to Security Group:
    Click "Security" tab in instance details
    Click the Security Group link (looks like sg-0abc123...)

STEP 3 - Edit inbound rules:
    Click "Inbound rules" tab
    Click "Edit inbound rules" button

STEP 4 - Add Grafana rule:
    Click "Add rule"
    Type:        Custom TCP
    Port:        3000
    Source:      My IP  (select from dropdown)
    Description: Grafana Dashboard

STEP 5 - Add Prometheus rule:
    Click "Add rule"
    Type:        Custom TCP
    Port:        9090
    Source:      My IP  (select from dropdown)
    Description: Prometheus

STEP 6 - Save:
    Click "Save rules"

NOTE: If your IP changes (home/office network change), you must
      come back here, delete old rules, and add new "My IP" rules.


--------------------------------------------------------------------------------
PHASE 9 - CONFIGURE GRAFANA (First Time Login)
--------------------------------------------------------------------------------

STEP 1 - Open in browser:

    http://your-server-ip:3000

STEP 2 - Login:

    Username: admin
    Password: admin
    (it will ask you to set a new password immediately)

STEP 3 - Add Prometheus as data source:

    Go to: Configuration -> Data Sources -> Add data source
    Choose: Prometheus
    URL: http://localhost:9090
    Click: Save & Test
    (should show green: Data source is working)

STEP 4 - Add Loki as data source:

    Go to: Configuration -> Data Sources -> Add data source
    Choose: Loki
    URL: http://localhost:3100
    Click: Save & Test
    (should show green: Data source connected)

STEP 5 - Import pre-built dashboards:

    Go to: Dashboards -> Import
    Enter ID and click Load, then Import

    Dashboard ID    What it shows
    ------------    ---------------------------------
    1860            Server metrics (CPU, RAM, Disk)
    9614            Nginx metrics
    11159           Node.js / PM2 app metrics


--------------------------------------------------------------------------------
DAILY MANAGEMENT COMMANDS
--------------------------------------------------------------------------------

Check all monitoring services:
    sudo systemctl status node_exporter prometheus loki promtail grafana-server

Restart a specific service:
    sudo systemctl restart prometheus
    sudo systemctl restart loki
    sudo systemctl restart grafana-server

View service logs if something fails:
    journalctl -u prometheus -n 50
    journalctl -u loki -n 50
    journalctl -u grafana-server -n 50

Check disk usage of monitoring data:
    du -sh /opt/monitoring/prometheus-data
    du -sh /opt/monitoring/loki-data


--------------------------------------------------------------------------------
TROUBLESHOOTING
--------------------------------------------------------------------------------

PROBLEM: curl: Failed to connect to localhost port 9100
FIX:
    sudo systemctl status node_exporter
    journalctl -u node_exporter -n 50
    ls /opt/monitoring/node_exporter-1.7.0.linux-amd64/node_exporter
    sudo systemctl restart node_exporter

PROBLEM: Loki fails with "mkdir" error
FIX:
    rm -rf /opt/monitoring/loki-data
    mkdir -p /opt/monitoring/loki-data/chunks
    mkdir -p /opt/monitoring/loki-data/rules
    mkdir -p /opt/monitoring/loki-data/index
    sudo systemctl restart loki

PROBLEM: Cannot open Grafana in browser (port 3000)
FIX:
    1. Check Grafana is running:  sudo systemctl status grafana-server
    2. Check AWS Security Group has port 3000 open for your IP
    3. Check your IP hasn't changed since adding the rule

PROBLEM: Grafana shows "Data source not working"
FIX:
    Make sure Prometheus/Loki are running:
        curl localhost:9090   (Prometheus)
        curl localhost:3100/ready   (Loki)
    Then retry Save & Test in Grafana

PROBLEM: No logs appearing in Grafana (Loki)
FIX:
    Check Promtail is running:
        sudo systemctl status promtail
    Check PM2 log path is correct:
        ls /root/.pm2/logs/
    Update path in /opt/monitoring/promtail-config.yml if needed
    Restart: sudo systemctl restart promtail


--------------------------------------------------------------------------------
ALTERNATIVE: GRAFANA CLOUD (if same-server install is not suitable)
--------------------------------------------------------------------------------

Use this if:
- Server RAM is less than 2GB
- You don't want to manage the monitoring tools yourself
- You want faster setup

Steps:
1. Sign up free at: https://grafana.com/auth/sign-up
2. Create a free stack (includes Prometheus + Loki hosted by Grafana)
3. Install only Promtail and Node Exporter on your server
4. Point them at your Grafana Cloud endpoints
5. Free tier: 10GB logs/month, 10,000 metrics/month

This removes need to manage Prometheus, Loki, and Grafana yourself.


--------------------------------------------------------------------------------
QUICK REFERENCE CARD
--------------------------------------------------------------------------------

Open Grafana:       http://your-server-ip:3000
Open Prometheus:    http://your-server-ip:9090

Start all:
    sudo systemctl start node_exporter prometheus loki promtail grafana-server

Stop all:
    sudo systemctl stop node_exporter prometheus loki promtail grafana-server

Restart all:
    sudo systemctl restart node_exporter prometheus loki promtail grafana-server

Check all status:
    sudo systemctl status node_exporter prometheus loki promtail grafana-server

================================================================================
END OF SKILL FILE
================================================================================
