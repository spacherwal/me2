# me2 — Claude Code Memory & Learnings

> Personal knowledge base auto-maintained by [Claude Code](https://claude.ai/code) across all active projects.

---

## Repository Structure

```
me2/
├── skills/                             # Custom Claude Code slash commands
│   ├── user-level/                     # ~/.claude/commands/ — available in all projects
│   │   ├── checkout-ui-review.md       # Review checkout screen code for UX issues
│   │   ├── figma-nextjs.md             # Figma URL → Next.js implementation
│   │   ├── ui-feedback-from-image.md  # Screenshot → structured UI/UX feedback
│   │   ├── prometheus-nextjs.md        # Prometheus + Grafana setup for Next.js
│   │   └── ec2-monitoring.md           # Full observability stack on Ubuntu EC2 (Node Exporter + prom-client + Prometheus + Loki + Promtail + Grafana)
│   │
│   └── D--Project-livguard-ecomm/     # Project-scoped skills
│       └── figma-livguard.md          # Livguard Figma file → code (file key pre-configured)
│
├── global/                             # User-level memory — applies across all projects
│   └── memory/
│       ├── MEMORY.md
│       ├── feedback_pincode_wms.md
│       ├── feedback_memory_lookup.md
│       ├── reference_me2_repo.md
│       └── project_livguard.md
│
├── D--Project-integration-service/     # Spring Boot integration microservice
│   └── memory/
│       ├── MEMORY.md                        # Index of all memories
│       ├── user_profile.md                  # User background & preferences
│       ├── feedback_preferences.md          # Coding & collaboration style
│       ├── feedback_compile_method.md       # Tooling preferences
│       ├── project_optimization_status.md  # Branch progress & rating
│       └── project_springboot3_upgrade.md  # SB 3.5.9 upgrade plan & notes
│
├── D--Project-lipl-scheduler/          # LIPL scheduler project
│
├── D--Project-livguard-d2c/        # LIPL D2C project
│   └── memory/
│       ├── MEMORY.md
│       ├── feedback_pincode_wms.md
│       ├── project_ltd2241.md
│       └── project_test_accounts.md
│
└── D--Project-livguard-ecomm/      # Livguard e-commerce (Next.js)
    └── memory/
        ├── MEMORY.md
        ├── feedback_ask_before_implementing.md
        ├── feedback_no_any_type.md
        ├── project_cart_context.md
        ├── project_products_page.md
        ├── project_session_apr15.md
        ├── project_session_apr16.md
        └── reference_me2_repo.md
```

---

## Projects

### 1. integration-service

Spring Boot microservice acting as an integration hub between multiple platforms.

#### Architecture Flow

```
                        ┌─────────────────────────────────────┐
                        │         integration-service          │
                        │           (Port 8082)                │
                        └──────────────┬──────────────────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              │                        │                        │
              ▼                        ▼                        ▼
     ┌────────────────┐    ┌──────────────────────┐   ┌────────────────┐
     │  Controllers   │    │  Scheduled Jobs      │   │    Webhooks    │
     │                │    │  (ExternalScheduler) │   │ /webhook/**    │
     │ /leadsquare    │    │                      │   │ (no auth)      │
     │ /leads         │    │  - lsqLeadOwners     │   └───────┬────────┘
     │ /razorpay      │    │  - lead_workflow      │           │
     │ /schedule      │    │  - lsqRazorpayAct.   │           │
     │ /sap           │    │  - razorpaySettlement │           │
     └───────┬────────┘    └──────────┬───────────┘           │
             │                        │                        │
             └────────────────────────┴────────────────────────┘
                                      │
                    ┌─────────────────┼──────────────────┐
                    │                 │                   │
                    ▼                 ▼                   ▼
          ┌──────────────┐  ┌──────────────────┐  ┌──────────────┐
          │  Services    │  │    MongoDB        │  │  Snowflake   │
          │              │  │  (Persistence)    │  │  (DWH Query) │
          │ LsqOwner     │  │                  │  └──────────────┘
          │ LsqActivity  │  │ APILogger        │
          │ RazorPay     │  │ LsqRazorpay      │
          │ SAP          │  │ LsqPaymentLink   │
          │ SolarLead    │  │ LeadsquaredUser  │
          │ SolarLsq     │  │ ScheduleConfig   │
          │ Email        │  │ ServiceLog       │
          │ AWS          │  └──────────────────┘
          └──────┬───────┘
                 │
     ┌───────────┼────────────┬────────────────┐
     │           │            │                │
     ▼           ▼            ▼                ▼
┌──────────┐ ┌────────┐ ┌─────────┐    ┌──────────┐
│LeadSquared│ │RazorPay│ │   SAP   │    │  AWS S3  │
│  (CRM)   │ │(Payments│ │  (ERP)  │    │ (Storage)│
└──────────┘ └────────┘ └─────────┘    └──────────┘
```

#### Security Flow

```
Incoming HTTP Request
        │
        ▼
  shouldNotFilter?
  /webhook/* ──► YES ──► Skip filter ──► Controller (no auth)
        │
        NO
        ▼
  Read X-API-Key header
        │
     match?
    ┌───┴────┐
   YES       NO
    │         │
    ▼         ▼
 Set auth   401 Unauthorized
 context        (stop)
    │
    ▼
 Controller
```

#### Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Java 21 (upgrading from 17) |
| Framework | Spring Boot 3.5.9 (upgrading from 2.7.2) |
| Persistence | MongoDB (Spring Data) |
| Data Warehouse | Snowflake (JDBC) |
| Build | Gradle |
| Security | Spring Security 6 — API Key filter |
| Logging | Log4j2 |
| Infra | AWS CodeDeploy → systemd on EC2 |
| CI/CD | Bitbucket Pipelines → S3 → CodeDeploy |

#### Code Quality Progress

```
Before optimization          After optimization
─────────────────────        ──────────────────
Field injection (@Autowired) Constructor injection (@RequiredArgsConstructor)
throws Exception everywhere  IntegrationException (RuntimeException)
407-line god class           Split → LsqOwnerService + LsqActivityService
Magic strings scattered      LeadSquaredApiConstants
Credentials logged in plain  sanitizeUrl() — redacted before MongoDB log
Raw Map<String,Object> DTOs  Typed DTOs (LsqBulkLeadUpdateRequest etc.)
Dead code (LivsolService)    Removed
No centralized error handler GlobalExceptionHandler (@RestControllerAdvice)

Rating: 6.5/10  ──────────────────────────────► 8.1/10
```

---

### 2. lipl-scheduler

LIPL internal scheduler project — session notes in conversation files.

---

### 3. livguard-d2c

LIPL D2C platform — memory covers pincode/WMS integration and test account setup.

---

### 4. livguard-ecomm

Livguard e-commerce storefront built with Next.js 16 App Router + React 19.

#### Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16.2.2 (App Router) |
| UI | React 19.2.4 |
| Language | TypeScript (strict mode) |
| Styling | CSS Modules |
| Auth | Cookie-based token via `/api/auth/set-token` |
| State | React Context (UserContext, CartContext) |

#### Key Architecture Decisions

- **CartContext** is the single source of truth for cart data — exposes `cartData`, `itemCount`, `addToCart`, `refreshCart`. Cart page and navbar both consume from `useCart()` — no duplicate API calls.
- **Auth-aware cart** — CartContext subscribes to `isAuthenticated` from UserContext; fetches cart on login, clears on logout without an API call.
- **Cart empty state** — API returns `statuscode: 204 / status: "NO_CONTENT"` for empty cart (not 200 with empty array). Always handle both cases.

---

## Skills

Custom slash commands (`/skill-name`) built for Claude Code. Install by copying to `~/.claude/commands/` (user-level) or `.claude/commands/` inside a project (project-level).

### User-level (all projects)

| Skill | Usage | Description |
|-------|-------|-------------|
| `checkout-ui-review` | `/checkout-ui-review` | Reads checkout code and flags UX issues: exposed IDs, dead buttons, missing progress indicators, mobile CTA placement, payment pre-selection sync, image error loops |
| `figma-nextjs` | `/figma-nextjs <figma-url>` | Full Figma → Next.js workflow: fetches design data, extracts tokens, handles Tailwind v3/v4, Framer Motion types, App Router conventions |
| `ui-feedback-from-image` | `/ui-feedback-from-image <path>` | Reads a screenshot and gives structured feedback across 12 UX/UI areas: hierarchy, typography, color, spacing, navigation, CTAs, forms, states, trust, mobile, IA, copy. Ends with a top-5 priority list. |
| `prometheus-nextjs` | `/prometheus-nextjs` | Full Prometheus + Grafana setup for Next.js: `prom-client` integration, `/api/metrics` route, `instrumentation.ts`, Prometheus scrape config, Grafana data source + dashboard import, optional Loki for logs, troubleshooting table |
| `ec2-monitoring` | `/ec2-monitoring` | Complete observability stack on Ubuntu EC2: Node Exporter (host metrics) + prom-client (app metrics) + Prometheus + Loki + Promtail + Grafana — includes architecture flowgraph, all config files, systemd unit files, dashboard IDs, and EC2 security group port reference |

### Project-level (livguard-ecomm only)

| Skill | Usage | Description |
|-------|-------|-------------|
| `figma-livguard` | `/figma-livguard` | Livguard-specific Figma → code. File key pre-configured (`7AW0hKHcyl9yZUOvRYSA8j`), component map included, design tokens resolved, known quirks documented |

---

## How This Works

Claude Code maintains a persistent memory system across sessions:

```
Session 1 ──► learns context ──► writes memory files
Session 2 ──► reads memory   ──► picks up where left off
Session N ──► updates memory ──► continuous knowledge base
```

Memory types stored:
- **user** — role, preferences, technical background
- **feedback** — what to do / avoid in future sessions
- **project** — ongoing work, decisions, status
- **reference** — where to find things in external systems

---

*Maintained by Claude Sonnet 4.6*
