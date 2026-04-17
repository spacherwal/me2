---
name: nextjs-ubuntu-aws-deploy
description: >
  Step-by-step guide to deploy a Next.js app from a Bitbucket repo onto an Ubuntu
  server sitting behind an AWS Load Balancer (ALB), with Nginx as reverse proxy and
  PM2 as process manager. Use this skill whenever you need to: set up a new QA/staging/
  production server for a Next.js project, re-deploy after a server rebuild, onboard a
  new server to an existing AWS Load Balancer, or troubleshoot 502 Bad Gateway errors
  on a Next.js deployment. Covers full flow: SSH key setup → clone → env vars → build
  → PM2 → Nginx → AWS Target Group registration → SSL.
---

# Next.js Deployment on Ubuntu + AWS Load Balancer

A complete, battle-tested guide for deploying a Next.js app from Bitbucket to an
Ubuntu server, served through an AWS Application Load Balancer.

---

## Architecture Overview

```
User → DNS (Route 53) → AWS Load Balancer (ALB) → EC2 Ubuntu Server
                                                         │
                                                    Nginx :80/:443
                                                         │
                                                   Next.js app (PM2)
                                                      :3000 or :8080
```

> ⚠️ CRITICAL LESSON LEARNED: The port your Next.js app runs on MUST match
> the port configured in the AWS Target Group. Mismatch = 502 Bad Gateway.

---

## Prerequisites Checklist

Before starting, confirm you have:
- [ ] SSH access to the Ubuntu server
- [ ] Bitbucket repo access (admin or write)
- [ ] AWS Console access (EC2 + Route 53 + Load Balancer)
- [ ] Domain DNS managed via Route 53
- [ ] `.env.local` values (API URLs, tokens, secrets)

---

## PHASE 1 — Server Setup

### Step 1 — Connect to Server
```bash
ssh your-user@your-server-ip
```

### Step 2 — Install Node.js via NVM
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
nvm alias default 20
node -v && npm -v   # verify
```

### Step 3 — Install PM2
```bash
npm install -g pm2
```

### Step 4 — Install Nginx
```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## PHASE 2 — Bitbucket SSH Setup

### Step 5 — Generate SSH Key on Server
```bash
ssh-keygen -t ed25519 -C "deploy@your-domain.com"
# Press Enter for all prompts (no passphrase)
cat ~/.ssh/id_ed25519.pub   # copy this output
```

### Step 6 — Add Key to Bitbucket
1. Bitbucket → Repository Settings → Access Keys → Add Key
2. Paste the public key, give it **Read** access
3. Test connection:
```bash
ssh -T git@bitbucket.org
```
Expected response (this is SUCCESS, not an error):
```
You can use git to connect to Bitbucket. Shell access is disabled.
```

---

## PHASE 3 — Clone & Configure

### Step 7 — Clone Repository
```bash
sudo mkdir -p /var/www/your-app
sudo chown $USER:$USER /var/www/your-app
git clone git@bitbucket.org:<workspace>/<repo-name>.git /var/www/your-app
cd /var/www/your-app
```

### Step 8 — Create Environment File
```bash
nano /var/www/your-app/.env.local
```
Add your environment variables:
```env
API_URL=https://your-api-host.example.com
API_TOKEN=Bearer <your-token>
NEXT_PUBLIC_API_URL=https://your-api-host.example.com
NEXT_PUBLIC_API_TOKEN=Bearer <your-token>
```
Save: `Ctrl+O` → `Ctrl+X`

---

## PHASE 4 — Build & Run

### Step 9 — Install Dependencies
```bash
cd /var/www/your-app
rm -rf node_modules          # clean slate
npm install
```

Verify `next` binary exists:
```bash
ls node_modules/.bin/next    # must return a path
```

### Step 10 — Build the App
```bash
npm run build
```

If `next: not found` error:
```bash
npm install next react react-dom
npm run build
```

Successful build output looks like:
```
✓ Compiled successfully
Route (app)    Size    First Load JS
...
```

### Step 11 — Decide the Port

> ⚠️ IMPORTANT: Pick a port and use it consistently everywhere.
> You need to match this port in: PM2 start command + Nginx config + AWS Target Group

Common choices: `3000` (Next.js default) or `8080`

### Step 12 — Start with PM2
```bash
cd /var/www/your-app

# Start on your chosen port (e.g., 8080)
pm2 start node_modules/.bin/next --name "your-app-name" -- start -p 8080

# Verify running
pm2 status
curl localhost:8080    # should return HTML
```

### Step 13 — Save PM2 for Reboots
```bash
pm2 save
pm2 startup
# Copy and run the command it outputs — looks like:
# sudo env PATH=$PATH:/root/.nvm/versions/node/v20.x.x/bin pm2 startup systemd -u root --hp /root
```

---

## PHASE 5 — Nginx Configuration

### Step 14 — Create Nginx Config
```bash
sudo nano /etc/nginx/sites-available/your-app
```

Paste (replace port and domain name):
```nginx
server {
    listen 80;
    server_name qa.yourdomain.com;   # ← exact domain, no typos

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    location / {
        proxy_pass http://localhost:8080;   # ← must match PM2 port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    location /_next/static/ {
        proxy_pass http://localhost:8080;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }
}
```

### Step 15 — Enable & Test Nginx
```bash
sudo ln -s /etc/nginx/sites-available/your-app /etc/nginx/sites-enabled/
sudo nginx -t                        # must say: syntax is ok
sudo systemctl restart nginx
```

---

## PHASE 6 — AWS Load Balancer Setup

> This phase is needed when DNS (Route 53) points to a Load Balancer, not directly
> to your EC2 instance. Signs of this: `nslookup` returns 2 IPs, or IPs don't match
> your server's `curl ifconfig.me` output.

### Step 16 — Verify IP Mismatch (Diagnosis)
```bash
# On server
curl ifconfig.me              # your server IP

# On local machine
nslookup qa.yourdomain.com   # DNS-resolved IP(s)
```
If IPs differ → you need to register your server with the Load Balancer.

### Step 17 — Register EC2 in Target Group
1. AWS Console → EC2 → **Target Groups**
2. Find the Target Group attached to your Load Balancer
3. **Targets** tab → **Register Targets**
4. Select your EC2 instance
5. Set **Port** → `8080` (must match your app port ⚠️)
6. Click **Include as pending** → **Register pending targets**

### Step 18 — Configure Health Check
In Target Group → **Health checks** tab → Edit:

| Setting | Value |
|---------|-------|
| Protocol | HTTP |
| Path | `/` |
| Port | `8080` (same as app port) |
| Healthy threshold | 2 |
| Interval | 30 seconds |

### Step 19 — Add Listener Rule
1. EC2 → **Load Balancers** → your ALB
2. **Listeners** tab → View/Edit Rules (port 80 or 443)
3. Add rule:

| Condition | Action |
|-----------|--------|
| Host header = `qa.yourdomain.com` | Forward to your Target Group |

### Step 20 — Update Security Group
EC2 → **Security Groups** → your EC2's security group → **Inbound Rules** → Add:

| Type | Port | Source |
|------|------|--------|
| Custom TCP | `8080` | Load Balancer's Security Group ID |

### Step 21 — Verify Target Health
Target Group → **Targets** tab → wait 1-2 mins → status should show **healthy** ✅

---

## PHASE 7 — SSL (HTTPS)

### Step 22 — Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d qa.yourdomain.com
# Follow prompts → choose "Redirect HTTP to HTTPS"
```

Test auto-renewal:
```bash
sudo certbot renew --dry-run
```

---

## PHASE 8 — Bitbucket Pipeline (Auto Deploy)

### Step 23 — Add SSH Key to Pipeline
1. Bitbucket → Repository Settings → Pipelines → **SSH Keys**
2. Click **Generate Keys**
3. Copy Public Key → add to server:
```bash
nano ~/.ssh/authorized_keys
# Paste on new line, save
```
4. Back in Bitbucket → Known Hosts → add `qa.yourdomain.com` → Fetch → Add Host

### Step 24 — Add Repository Variables
Bitbucket → Repository Settings → Pipelines → **Repository Variables**:

| Variable | Secured? |
|----------|----------|
| `SERVER_HOST` | No |
| `SERVER_USER` | No |
| `DEPLOY_PATH` | No |
| `API_URL` | ✅ Yes |
| `API_TOKEN` | ✅ Yes |
| `NEXT_PUBLIC_API_URL` | ✅ Yes |
| `NEXT_PUBLIC_API_TOKEN` | ✅ Yes |

### Step 25 — Create `bitbucket-pipelines.yml`
Place at repo root:

```yaml
image: node:20

definitions:
  caches:
    nextjs: .next/cache

  steps:
    - step: &build
        name: Install Dependencies & Build
        caches:
          - node
          - nextjs
        script:
          - |
            cat > .env.local <<EOF
            API_URL=$API_URL
            API_TOKEN=$API_TOKEN
            NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
            NEXT_PUBLIC_API_TOKEN=$NEXT_PUBLIC_API_TOKEN
            EOF
          - npm ci
          - npm run build
        artifacts:
          - .next/**
          - node_modules/**
          - public/**
          - package.json
          - package-lock.json
          - .env.local

    - step: &deploy-qa
        name: Deploy to QA Server
        deployment: qa
        script:
          - pipe: atlassian/ssh-run:0.4.1
            variables:
              SSH_USER: $SERVER_USER
              SERVER: $SERVER_HOST
              COMMAND: >
                set -e &&
                cd $DEPLOY_PATH &&
                git pull origin main &&
                cat > .env.local <<EOF
                API_URL=$API_URL
                API_TOKEN=$API_TOKEN
                NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
                NEXT_PUBLIC_API_TOKEN=$NEXT_PUBLIC_API_TOKEN
                EOF
                npm ci --omit=dev &&
                npm run build &&
                pm2 reload your-app-name --update-env &&
                echo "✅ Deployment complete!"

pipelines:
  branches:
    main:
      - step: *build
      - step: *deploy-qa
    develop:
      - step:
          name: Build Check (develop)
          caches:
            - node
            - nextjs
          script:
            - npm ci
            - npm run build

  custom:
    deploy-to-qa:
      - step: *build
      - step: *deploy-qa
```

---

## Troubleshooting Guide

### 502 Bad Gateway
Run through this checklist in order:

```bash
# 1. Is Next.js running?
pm2 status
curl localhost:8080

# 2. Is Nginx running?
sudo systemctl status nginx

# 3. Does Nginx config have correct port and domain?
sudo cat /etc/nginx/sites-available/your-app

# 4. Is symlink present?
ls -la /etc/nginx/sites-enabled/

# 5. Nginx error log
sudo tail -20 /var/log/nginx/error.log

# 6. Does server IP match DNS?
curl ifconfig.me
# compare with: nslookup qa.yourdomain.com (run locally)
```

Most common 502 causes:
- App not running (`pm2 status` shows stopped/errored)
- Port mismatch between app, Nginx, and AWS Target Group ← most common!
- `server_name` typo in Nginx config
- EC2 not registered in AWS Target Group
- Security Group blocking Load Balancer → EC2 traffic

### `next: not found` during build
```bash
rm -rf node_modules
npm install
npm run build
```

### App dies after terminal closes
You used `npm start` directly. Switch to PM2:
```bash
pm2 start node_modules/.bin/next --name "your-app-name" -- start -p 8080
pm2 save
pm2 startup
```

### `nginx.service is not active, cannot reload`
```bash
sudo systemctl start nginx   # start first
sudo systemctl reload nginx  # then reload
```

### SSH to Bitbucket says "Shell access is disabled"
This is SUCCESS ✅ — not an error. Proceed with `git clone`.

---

## Manual Redeploy (without pipeline)
```bash
cd /var/www/your-app
git pull origin main
npm install
npm run build
pm2 reload your-app-name --update-env
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Check app status | `pm2 status` |
| View live logs | `pm2 logs your-app-name` |
| Restart app | `pm2 restart your-app-name` |
| Reload (zero downtime) | `pm2 reload your-app-name` |
| Stop app | `pm2 stop your-app-name` |
| Nginx syntax check | `sudo nginx -t` |
| Restart Nginx | `sudo systemctl restart nginx` |
| Nginx error log | `sudo tail -f /var/log/nginx/error.log` |
| Get server IP | `curl ifconfig.me` |
| Check port in use | `sudo lsof -i :8080` |
