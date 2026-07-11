# 📚 Knowledge — cross-project learnings & playbooks

Free-form, non-project knowledge. Reusable patterns, reference architectures, and hard-won ops
landmines that span more than one repo. Grows over time — add an entry whenever a learning outlives
the project it came from.

## Index
- [`livguard-fargate-apigw-deployment.md`](livguard-fargate-apigw-deployment.md) — the ECS Fargate +
  API Gateway (VPC Link → Cloud Map) deployment pattern shared by `livguard-integration` and
  `livguard-distribution-network`, and the landmines that bite. **Ops-critical.**
- [`cicd-bitbucket-codedeploy-ec2.md`](cicd-bitbucket-codedeploy-ec2.md) — the older Bitbucket→S3→
  CodeDeploy→EC2/systemd pipeline pattern (integration-service, schedulers, reports). *Stub.*
- [`reference-architecture-integration-service.md`](reference-architecture-integration-service.md) —
  annotated architecture / security / code-quality diagrams for the integration hub (rescued from
  the old README).

## To add (stubs worth filling as the need arises)
- Spring Boot 2→3 and 3→4 / Java 17→21→25 migration checklist (source: integration-service SB3
  upgrade memory + livguard-integration Java 25 build).
- Grails 6 monolith conventions distilled (source: `lipl-grails3` CLAUDE.md).
- Snowflake data-ops patterns (MERGE-as-dedup, pincode dedup, VARCHAR coord repair) — source:
  livguard-integration.
