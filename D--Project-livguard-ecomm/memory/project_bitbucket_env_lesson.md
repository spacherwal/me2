---
name: Bitbucket Pipeline .env.local Gap
description: QA env vars — the pipeline heredoc must list every variable the app needs; missing vars silently become empty strings
type: project
originSessionId: c8b1da0f-fc98-496b-a149-091728ef9216
---
The Bitbucket pipeline writes `.env.local` via a heredoc in the build step. If a variable is not listed in that heredoc (even if it exists as a Bitbucket repo variable), it is absent from the deployed `.env.local`.

**Why:** The `cat > .env.local <<EOF ... EOF` block in `bitbucket-pipelines.yml` is the only source of `.env.local` on QA — the EC2 server never gets it any other way. Missing entries silently produce empty values (`API_URL=`), which breaks `axios.create({ baseURL })` and all downstream API calls.

**How to apply:** Whenever a new env var is added to `.env.local` locally, also add it to:
1. The Bitbucket repository variables (Settings → Repository variables)
2. The `cat > .env.local <<EOF` block inside `bitbucket-pipelines.yml`

Variables used server-side (no `NEXT_PUBLIC_` prefix) still need to be in `.env.local` on the server for Next.js to pick them up at runtime.
