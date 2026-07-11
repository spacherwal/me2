# Reference architecture — integration-service

Rescued from the old repo README. `integration-service` is the **older-generation** integration hub
(Spring Boot 2.7 / Java 17, MongoDB + Snowflake), deployed via the Bitbucket→S3→CodeDeploy→EC2
pattern (see [`cicd-bitbucket-codedeploy-ec2.md`](cicd-bitbucket-codedeploy-ec2.md)) — distinct from
the newer Fargate services. Kept here as a reference example of a clean integration-hub layout.

## Architecture flow

```
                        ┌─────────────────────────────────────┐
                        │         integration-service          │
                        │           (Port 8082)                │
                        └──────────────┬──────────────────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              ▼                        ▼                        ▼
     ┌────────────────┐    ┌──────────────────────┐   ┌────────────────┐
     │  Controllers   │    │  Scheduled Jobs      │   │    Webhooks    │
     │ /leadsquare    │    │  (ExternalScheduler) │   │ /webhook/**    │
     │ /leads         │    │  - lsqLeadOwners     │   │ (no auth)      │
     │ /razorpay      │    │  - lead_workflow      │   └───────┬────────┘
     │ /schedule      │    │  - lsqRazorpayAct.   │           │
     │ /sap           │    │  - razorpaySettlement │           │
     └───────┬────────┘    └──────────┬───────────┘           │
             └────────────────────────┴────────────────────────┘
                                      │
                    ┌─────────────────┼──────────────────┐
                    ▼                 ▼                   ▼
          ┌──────────────┐  ┌──────────────────┐  ┌──────────────┐
          │  Services    │  │    MongoDB        │  │  Snowflake   │
          │ Lsq* / SAP   │  │  (Persistence)    │  │  (DWH Query) │
          │ RazorPay/AWS │  │  APILogger etc.   │  └──────────────┘
          └──────┬───────┘  └──────────────────┘
                 ▼
   LeadSquared (CRM) · RazorPay (Payments) · SAP (ERP) · AWS S3
```

## Security flow (API-key filter, webhook bypass)

```
Incoming HTTP Request
        │
        ▼
  shouldNotFilter?  /webhook/* ──► YES ──► skip filter ──► Controller (no auth)
        │ NO
        ▼
  Read X-API-Key header ──► match? ── YES ─► set auth context ─► Controller
                                   └─ NO ──► 401 Unauthorized
```

## Code-quality overhaul (the refactor that took it 6.5 → 8.1/10)

```
Before                          After
─────────────────────────       ──────────────────────────────────────────
Field injection (@Autowired)     Constructor injection (@RequiredArgsConstructor)
throws Exception everywhere      IntegrationException (RuntimeException)
407-line god class               Split → LsqOwnerService + LsqActivityService
Magic strings scattered          LeadSquaredApiConstants
Credentials logged in plain      sanitizeUrl() — redacted before MongoDB log
Raw Map<String,Object>           Typed DTOs
No centralized error handler     GlobalExceptionHandler (@RestControllerAdvice)
Dead code (LivsolService)        Removed
No tests                         Flapdoodle embedded MongoDB, 13 test files
```

Full change log and the stashed SB2→SB3 upgrade notes live in the project memory:
`D--Project-integration-service/memory/`.
