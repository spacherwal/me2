# Playbook — Bitbucket → S3 → CodeDeploy → EC2 (STUB)

The older-generation deploy pattern for JVM services that run on EC2 (not Fargate):
`integration-service`, `lipl-scheduler`, `livsol-report`, `mis-report-rest-api`. Contrast with the
newer [Fargate + API Gateway pattern](livguard-fargate-apigw-deployment.md).

> **Stub — flesh out from a real `bitbucket-pipelines.yml` + `appspec.yml` when next in one of these
> repos.** There's also a `cicd_skill` / `setup-nextjs-cicd` skill installed that scaffolds this exact
> stack — reuse it rather than hand-rolling.

## Shape

```
Bitbucket Pipelines ──build──► artifact ──► S3 bucket ──► AWS CodeDeploy ──► EC2 (systemd unit)
                                                                              e.g. ltd-integration.service
```

## To capture here
- [ ] `bitbucket-pipelines.yml` steps (build, test gate, upload, trigger CodeDeploy).
- [ ] `appspec.yml` + the lifecycle hook scripts (`BeforeInstall` / `ApplicationStart` /
      `ValidateService`), and where the systemd unit is defined.
- [ ] S3 bucket + IAM/deployment-group setup (buckets seen: `ltd-cicd`, `ltd-cicd-prod`).
- [ ] Slack deploy-notification wiring.
- [ ] Rollback story on EC2 (how a bad revision is reverted vs. the Fargate SHA-pin approach).

## Known facts
- Free-tier note carried from other work: prefer OTA/JS-only updates where possible to avoid burning
  full rebuilds (applies to the Expo project, not these — noted for cross-reference).
- The Grails monolith (`lipl-grails3`) uses **CircleCI**, not Bitbucket, for this same S3→CodeDeploy→
  EC2/Tomcat flow — a separate pipeline worth its own notes.
