---
name: CICD Debugging Lesson Apr 21 2026
description: Root causes and fixes for CodeDeploy pipeline failures on EC2 Ubuntu — bootstrap loop and npm ci lock file mismatch
type: project
originSessionId: 676a460f-7e6d-46b6-b313-438df0c0671a
---
# CI/CD Debugging Lesson — Apr 21 2026

## Setup
Bitbucket Pipelines → S3 → AWS CodeDeploy → EC2 Ubuntu (ap-south-1)
App: livguard-ecomm | Instance: i-04cbb94221877cac3 | CodeDeploy app: Livguard-Ecomm-Staging / Livguard-Ecomm-Staging-Group

---

## Problem 1 — Deployment Bootstrap Loop

**Error:** "CodeDeploy agent was not able to receive the lifecycle event"

**Why:** During `ApplicationStop`, CodeDeploy runs scripts from the *previous* deployment's archive on disk. If the previous deployment failed and was cleaned up, that directory no longer exists. The agent reports it can't receive the event — misleading wording for a missing prior archive.

**Fix:** Add `--ignore-application-stop-failures` to `aws deploy create-deployment` in `bitbucket-pipelines.yml`. Safe long-term — once a deployment succeeds, future `ApplicationStop` hooks run normally.

**How to apply:** Any time a fresh EC2 instance is added to a deployment group, or after a streak of failed deployments cleaned up by CodeDeploy, this flag is needed to break the loop.

---

## Problem 2 — npm ci Fails on EC2

**Error:** `npm ci can only install packages when your package.json and package-lock.json are in sync. Missing: prom-client, @opentelemetry/api, tdigest, bintrees`

**Why:** Packages were added to `package.json` manually without running `npm install` to update the lock file. The stale `package-lock.json` was committed. `npm ci` strictly requires them to be in sync.

**Fix:** Run `npm install` locally → commit updated `package-lock.json` → push to release.

**How to apply:** Enforce as a team rule — never commit `package.json` changes without also committing the regenerated `package-lock.json` in the same commit.

---

## Diagnostic Path Taken

1. Confirmed agent running: `sudo service codedeploy-agent status`
2. Confirmed network/IAM OK: agent logs showed `poll_host_command` returning 200
3. Confirmed EC2 tags (`Name: lipl-qa-ubuntu`) matched deployment group tag filter — not the issue
4. Identified bootstrap loop from agent log: `commandName: ApplicationStop` referencing cleaned-up previous deployment directory
5. Fixed with `--ignore-application-stop-failures`
6. Next failure: `AfterInstall` → `install_deps.sh` → `npm ci` lock file mismatch
7. Full error visible in `/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log`

---

## Known Process Gaps (to improve)

- Build (`npm ci` + `npm run build`) runs on EC2, not in CI — slow, no rollback if build fails
- No `--auto-rollback-configuration` on CodeDeploy deployment
- `validate.sh` uses `sleep 10` (fragile) instead of a retry loop
- `AllAtOnce` deployment config — risky if scaling to multiple EC2s
- Zip root verification too loose (`grep appspec.yml` matches nested paths)
