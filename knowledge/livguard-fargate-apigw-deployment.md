# Livguard — ECS Fargate + API Gateway deployment pattern & landmines

Shared deployment model for the newer Livguard services (`livguard-integration` — SB4/Java25 backend;
`livguard-distribution-network` — Next.js 16 frontend). Both deliberately mirror each other. This is
the reusable pattern plus the traps found the hard way. Region `ap-south-1`.

## Architecture (no ALB, no NAT)

```
Client → API Gateway (HTTP API) → VPC Link → Cloud Map (SRV) → ECS Fargate task IPs
```

- **Cloud Map records must be SRV, not A.** SRV carries the port (apps listen on 3000); A records
  default to 80 and silently break the VPC Link integration.
- **API Gateway needs both `ANY /` and `ANY /{proxy+}` routes.** `/{proxy+}` alone doesn't match the
  bare root path — `/` 404s while everything else works.
- **API Gateway has a hard 29s integration timeout.** Long/bulk ops return 504. Design bulk endpoints
  idempotent (Snowflake `MERGE` is) so a 504 is safe to retry.

## Terraform vs deploy.sh — who owns what

- ECS service sets `ignore_changes = [task_definition]`. **Terraform owns the task-def *template*;
  `deploy.sh`/CI own the *rollout*.**
- **After any `terraform apply` that changes the task def (env/secrets), you MUST run `./deploy.sh`**
  or the change never rolls out. `deploy.sh` bases each new revision on the latest ACTIVE one, so TF
  env/secret edits carry over.

## Rollback depends on SHA-pinned revisions

- `distribution-network` registers **SHA-pinned** task-def revisions (not `:latest`, not
  `--force-new-deployment`). That's what makes the deployment circuit-breaker rollback meaningful —
  the previous revision points at the previous code SHA.
- **The sibling `livguard-integration` deploys `:latest`, so its rollback is a no-op.** Do NOT
  "align" the frontend to the sibling's `:latest` approach — you'd silently disable real rollback.

## Landmines (each one deploys GREEN)

- **ARM64/amd64 mismatch (`livguard-integration`):** `hosting_plan.md`, `docs/aws-architecture.md`,
  and the Terraform `runtimePlatform` say **ARM64**, but the committed `deploy.sh` builds
  `--platform linux/amd64`. An amd64 image against an ARM64 task def fails to launch (exec-format).
  Reconcile before the next prod deploy. (`distribution-network` is correctly all-x86_64 in lockstep
  — task def, deploy.sh, and Bitbucket runners all amd64; change them together or tasks die.)
- **Wrong backend URL deploys GREEN:** the container health check only probes the app's own
  `/api/health`, never the backend. TF validation catches a *placeholder* tfvar but not a
  wrong-but-valid URL. Symptom of misconfig = blank UI / empty data, no error.
- **Proxy routes return empty, not mock:** `distribution-network`'s `app/api/*` return
  `{data:[],count:0}` on any error or missing env — by design (mock fallback was removed). README
  claiming "falls back to mock data" is stale.
- **`trustHost: true` is load-bearing** (Auth.js v5, hardcoded in `auth.ts` — deliberately not the
  env var). Without it, Auth.js throws `UntrustedHost` behind the proxy and the guard **fails open**
  — the dashboard renders for anonymous users. Never remove it.
- **Default admin seeded everywhere:** `livguard-integration`'s `AdminSeeder` creates
  `admin@livguard.com` / `password123` on first boot in **all** profiles, behind the public
  `/api/auth/validate`. Change immediately on any new environment.

## Accounts & guards

- Dev + stage share AWS account `521577804533` (profile `LTD`). **Prod is a separate account.**
- Guarded twice: Terraform `check "account_guard"` (fails at plan) and `deploy.sh`
  `sts get-caller-identity` verification. ECR repo + CI IAM user are created once per account.
- Cost control: scheduled **scale-to-0** on the backend (up at :00 every 2h UTC, down at :20) → 503
  outside the windows. CI creds are Bitbucket deployment-environment-scoped.

## CI branch reality (correct the docs)

- `distribution-network` auto-deploys **dev on branch `feature/business-enhancements`**, not
  `master` — despite README and CLAUDE.md both saying "every master commit." Prod is a manual
  `deploy-prod` custom pipeline.

## Next.js 16 caveat (frontend)

- Next.js 16 has breaking changes vs. common knowledge — middleware is `proxy.ts` (renamed), Sentry
  client init is `instrumentation-client.ts`. `AGENTS.md` mandates reading
  `node_modules/next/dist/docs/` before framework-level edits. All `/api/*` routes bypass the auth
  guard by design — do not add auth middleware to them.
