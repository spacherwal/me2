---
name: hosting-plan
description: Planned AWS ECS Fargate deployment — decisions made and open questions
metadata: 
  node_type: memory
  type: project
  originSessionId: 3d98ac4e-7410-4d1b-a9b9-17c2bb64c881
---

Target: Docker container on **Amazon ECS Fargate**, publicly reachable via **ALB** (inbound push from third party).

**Decisions confirmed:**
- ECS Fargate (not EC2 launch type) — no cluster node management
- ALB as public entry point (inbound push requires stable reachable endpoint)
- HTTP for now (no domain yet); HTTPS via ACM when domain is acquired
- Starting from scratch — no existing AWS account/VPC
- IaC approach: **not yet decided** — pending user choice between Terraform vs AWS Console

**AWS resources to provision:**
- VPC — 2 public subnets (ALB) + 2 private subnets (ECS tasks)
- Internet Gateway + NAT Gateway
- ECR repository (Docker image storage)
- ECS Fargate cluster + task definition + service
- ALB (HTTP port 80 → container port 8083; HTTPS later)
- Secrets Manager — all 8 env vars (SNOWFLAKE_*, API_KEY_1, API_KEY_2)
- IAM roles — ECS task execution role with Secrets Manager access
- Security groups — ALB SG (public 80/443), ECS SG (inbound from ALB only)

**Deliverables planned:**
1. `Dockerfile` — multi-stage build, Java 25 (`eclipse-temurin:25-jre`)
2. Terraform configs (or Console steps, pending decision) for full stack
3. `deploy.sh` — ECR push + ECS service update flow

**Open questions to resolve next session:**
- Terraform vs AWS Console?
- AWS region preference?
- Domain/HTTPS timeline?

**Why:** Third-party system pushes inbound data to this Spring Boot app. App must be publicly reachable at a stable URL.

**How to apply:** When resuming this topic, pick up from the open questions above. Don't re-explain the architecture — jump straight to the pending IaC decision.
