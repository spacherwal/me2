# Prometheus + Grafana Setup for Next.js

Set up metrics collection and visualization for a Next.js app using Prometheus and Grafana.

## Usage

```
/prometheus-nextjs
```

Run from the root of any Next.js project.

---

## Overview

Prometheus scrapes metrics; Grafana visualizes them. Next.js does not expose metrics by default — you must add a `/api/metrics` endpoint using `prom-client`.

**Full stack:**
- `prom-client` — emits Prometheus-format metrics from Next.js
- Prometheus (port 9090) — scrapes and stores metrics
- Grafana (port 3000) — dashboards and visualization
- *(Optional)* Loki + Promtail — log aggregation (Prometheus is metrics-only)

---

## Step 1: Expose `/api/metrics` from Next.js

### Install prom-client

```bash
npm install prom-client
```

### Create the registry singleton

`src/lib/metrics.ts`:
```ts
import { Registry, collectDefaultMetrics } from 'prom-client';

const registry = new Registry();
registry.setDefaultLabels({ app: '<your-app-name>' });
collectDefaultMetrics({ register: registry });

export { registry };
```

### Initialize on server startup

`src/instrumentation.ts` (in `src/` root, not inside `app/`):
```ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./lib/metrics');
  }
}
```

> The `NEXT_RUNTIME` guard is required — `prom-client` does not support the Edge runtime.

### Create the route handler

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

Verify locally: `curl http://localhost:<port>/api/metrics` — should return lines starting with `# HELP` and `# TYPE`.

---

## Step 2: Configure Prometheus to scrape it

Edit `prometheus.yml` on the server. Add a scrape job under `scrape_configs`:

```yaml
scrape_configs:
  - job_name: 'nextjs-app'
    static_configs:
      - targets: ['localhost:<nextjs-port>']
    metrics_path: '/api/metrics'
    scrape_interval: 15s
```

Restart Prometheus:
```bash
sudo systemctl restart prometheus
```

Verify: open `http://<server-ip>:9090/targets` — the job should show **UP** in green.

If it shows **DOWN**, check:
- Is the Next.js app actually running on that port?
- Is port accessible from Prometheus (localhost binding)?
- Does `/api/metrics` return 200?

---

## Step 3: Add Prometheus as a Grafana data source

1. Open Grafana at `http://<server-ip>:3000`
2. **Connections → Data Sources → Add data source → Prometheus**
3. URL: `http://localhost:9090`
4. Click **Save & Test** — should return green confirmation

---

## Step 4: Import a dashboard

Option A — import a pre-built Node.js dashboard:
- In Grafana: **Dashboards → Import**
- Enter dashboard ID `11159` (Node.js Application Dashboard) or `13659`
- Select your Prometheus data source → Import

Option B — build custom panels using PromQL. Useful starter queries:

| Metric | PromQL |
|--------|--------|
| HTTP request rate | `rate(http_requests_total[5m])` |
| Node.js heap used | `nodejs_heap_size_used_bytes` |
| Event loop lag | `nodejs_eventloop_lag_seconds` |
| Active handles | `nodejs_active_handles_total` |
| Process CPU | `rate(process_cpu_seconds_total[1m])` |

---

## Step 5 (Optional): Add log visibility with Loki

Prometheus collects **metrics only** — not logs. For logs in Grafana:

1. Install Loki on the server
2. Install Promtail — configure it to tail your app's stdout or log files and ship to Loki
3. In Grafana: **Connections → Data Sources → Add → Loki**, URL `http://localhost:3100`
4. Use the **Explore** tab to query logs with LogQL

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `/api/metrics` returns 404 | `instrumentation.ts` not in `src/` | Move file to `src/instrumentation.ts` |
| `/api/metrics` returns 500 | prom-client running in Edge runtime | Add `NEXT_RUNTIME === 'nodejs'` guard |
| Prometheus target shows DOWN | Wrong port or app not running | Check `systemctl status` and port |
| Grafana shows "No data" | Data source not saved, or wrong URL | Re-save data source, check `localhost:9090` |
| Metrics appear but no panels | Dashboard not imported / wrong data source selected | Re-import dashboard, select correct source |
