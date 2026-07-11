# 🧠 me2 — Personal Knowledge Base & Learnings

> A free-form knowledge base maintained by [Claude Code](https://claude.ai/code) across all of
> Sahil Pacherwal's work. **Projects are one part of it** — the base also holds cross-cutting
> knowledge, playbooks, custom skills, and a technical biography, and grows over time.

---

## 📂 How this repo is organized

```
me2/
  README.md                  # this index — the navigation layer for the whole base
  tech-biography.md          # cross-cutting: the engineer's technical bio (résumé/self-review source)
  knowledge/                 # free-form, non-project learnings & playbooks (grows over time)
  skills/                    # custom Claude Code slash-command skills
    user-level/                  # ~/.claude/commands/ — available in all projects
    D--Project-<name>/           # project-scoped skills
  global/memory/             # user-level memory — applies across every project
  D--Project-<name>/         # per-project mirror of .claude/projects/<name>/
    <uuid>/, <uuid>.jsonl        # raw session transcripts (sync-managed — left untouched)
    memory/                      # curated, high-signal memory for that project
      MEMORY.md                      # one-line index
      {user,feedback,project,reference}_<slug>.md
```

Project folders keep their encoded `D--Project-*` names and stay flat, because they **mirror**
`.claude/projects/`. Domain grouping lives in this index only (below) — folders are never moved, so
the mirror never desyncs.

## 🧭 Two layers, one rule

- 📖 **Index / knowledge layer** (this README, `tech-biography.md`, `knowledge/`, per-project cards):
  human-facing documentation. Stack, versions, architecture, status — **derivable facts are welcome
  here.**
- 🧷 **`memory/` layer** (the typed `*.md` files Claude auto-recalls next session): **non-obvious
  only** — decisions, constraints, gotchas, feedback. Never restate what the repo already records
  (stack, structure, git history); that wastes recall context and rots. Source of truth for
  derivable facts is always the code.

**Memory types:** `user` (role/preferences) · `feedback` (do/avoid, with the why) · `project`
(ongoing work, decisions, status) · `reference` (where things live in external systems).

---

## 🗂️ The estate — grouped by domain

Employer **LIPL** (`lakshyanet.com`) builds the platforms behind **Livguard** (energy —
batteries/inverters/solar), **LockTheDeal / LTD** (marketplace), **Livguard Solar**, and **LivFin**
(NBFC lending).

**Status:** 🟢 Production · 🔵 Active · 🟡 MVP · ⚪ Learning · 📄 Docs · ❓ Unknown
**me2 presence:** ✅ memory (folder + curated memory) · 🪞 mirror (transcripts only) · 📇 index (this card only, no me2 folder yet)

### 🛒 1. Commerce — storefronts & marketplace
| Project | Role | Primary stack | Status | me2 |
|---|---|---|---|---|
| `lipl-grails3` | **B2B** e-commerce monolith (Grails full-stack) | Grails 6.1.2 / Groovy, MongoDB+GORM, Redis, ES6+Algolia+Atlas Search, Quartz | 🟢 | ✅ |
| `livguard-d2c` | **D2C/B2C** e-commerce backend (Grails; legacy GSP frontend being replaced) | Grails/GSP, Java, Vinculum WMS | 🟢 | ✅ |
| `livguard-ecomm` | D2C storefront — Next.js frontend segregated from the `livguard-d2c` Grails backend | Next.js 16 (App Router), React 19, TS, TanStack Query, Tailwind 4, Prometheus | 🟢 | ✅ |
| `livguardsolar-website` | Livguard Solar marketing + lead-gen site (no cart) | Remix 1.19, React 18, TS, PostgreSQL, JWT, Tailwind 3 | 🟢 | ✅ |
| `livguardsolar-website-plan-autonomous` | Planning/companion to solar site | Markdown plans, route/theme manifests | 📄 | 📇 |
| `product-app` | Static Livguard marketing demo (OTP/product code orphaned) | React 19, TS, Vite, react-router 7 | ⚪ | ✅ |

### 🗺️ 2. Distribution & field network
| Project | Role | Primary stack | Status | me2 |
|---|---|---|---|---|
| `livguard-distribution-network` | Distributor/retailer map (frontend) | Next.js 16, React 19, Leaflet+CARTO, Auth.js v5, Zustand, Sentry | 🟢 | ✅ |
| `livguard-integration` | Distributor data/bulk API (backend) | Spring Boot 4.0.6 / Java 25 (Gradle Kotlin DSL), Snowflake JDBC | 🟢 | ✅ |
| `livguard-distribution-docs` | Map enhancement tracker | Markdown | 📄 | 📇 |

### 🔌 3. Integration, scheduling & reporting
| Project | Role | Primary stack | Status | me2 |
|---|---|---|---|---|
| `integration-service` | Integration hub (CRM/payments/ERP/DWH) | Spring Boot 2.7 / Java 17, MongoDB, Snowflake, Gradle | 🟢 | ✅ |
| `lipl-scheduler` | Report/scheduler service | Spring Boot 2.7 / Java 17, MongoDB, Snowflake, POI | 🟢 | 🪞 |
| `livsol-report` | Report/scheduler service (fork of scheduler) | Spring Boot 2.7 / Java 17, MongoDB, Snowflake, POI | 🟢 | 📇 |
| `livmonitor-onboard` | "Livmonitor" onboarding service (early) | Spring Boot 4.0 / Java 25, MongoDB | 🔵 | 📇 |

### 🏦 4. Fintech — LivFin (NBFC lending)
| Module | Role | Primary stack | Status | me2 |
|---|---|---|---|---|
| `LivfinUserOnBoarding` | KYC/credit onboarding journey | Spring Boot 1.5 + Spring Cloud Dalston, JHipster, Java 8, MongoDB+Redis, Eureka/Config, Feign+Hystrix | 🟢 | 📇 |
| `LivfinGateway` | API gateway + admin UI | JHipster gateway, Spring Boot, Angular 4 | 🟢 | 📇 |
| `LivfinThirdParty` | ~55 KYC/credit integration WARs | Java WARs (Aadhaar/PAN/CIBIL/AML/GST/MCA/Perfios/eSign/video-KYC/liveness/BBPS…) | 🟢 | 📇 |
| `LivfinCore` | Shared utilities | Java 8, Maven, AWS SES/SNS/S3, PDFBox/iText | 🟢 | 📇 |
| `mis-report-rest-api` | MIS reporting API | Spring Boot, Maven | 🟢 | 📇 |
| `LivfinAdminWeb` | Admin web (not populated locally) | unknown | ❓ | 📇 |

### 📱 5. Personal / MVP
| Project | Role | Primary stack | Status | me2 |
|---|---|---|---|---|
| `rent-tracker-expo` | Personal rent/expense tracker (pivoting to shared cloud) | Expo SDK 54, React Native 0.81, expo-router 6, TS, expo-sqlite; EAS APK | 🔵 | ✅ |
| `nuts-warehouse` | Dry-fruit factory stock MVP | Node/Express, Prisma, PostgreSQL, JWT; React (Vite), Tailwind; docker-compose | 🟡 | 📇 |

### 🎓 6. Learning & demos
| Project | Role | Primary stack | Status | me2 |
|---|---|---|---|---|
| `jhipster-demo` | JHipster scaffold (skill refresh) | JHipster 8.11, Spring Boot 3.4, Java 17, React, PostgreSQL | ⚪ | 📇 |
| `nextjs-demo` | Team-onboarding demo | Next.js 14, React 18, TS | ⚪ | 📇 |
| `learn-node` | Node.js scratch/learning | Node.js (fs/async); also hosted a Figma→React experiment | ⚪ | ✅ |

---

## 🧰 Skills

Custom Claude Code slash commands (`/skill-name`). Install by copying to `~/.claude/commands/`
(user-level) or `.claude/commands/` inside a project (project-level).

### User-level (all projects)
| Skill | Usage | Description |
|-------|-------|-------------|
| `checkout-ui-review` | `/checkout-ui-review` | Reads checkout code and flags UX issues: exposed IDs, dead buttons, missing progress indicators, mobile CTA placement, payment pre-selection sync, image error loops |
| `figma-nextjs` | `/figma-nextjs <figma-url>` | Full Figma → Next.js workflow: fetches design data, extracts tokens, handles Tailwind v3/v4, Framer Motion types, App Router conventions |
| `ui-feedback-from-image` | `/ui-feedback-from-image <path>` | Reads a screenshot and gives structured feedback across 12 UX/UI areas: hierarchy, typography, color, spacing, navigation, CTAs, forms, states, trust, mobile, IA, copy. Ends with a top-5 priority list. |
| `prometheus-nextjs` | `/prometheus-nextjs` | Full Prometheus + Grafana setup for Next.js: `prom-client` integration, `/api/metrics` route, `instrumentation.ts`, Prometheus scrape config, Grafana data source + dashboard import, optional Loki for logs, troubleshooting table |
| `ec2-monitoring` | `/ec2-monitoring` | Complete observability stack on Ubuntu EC2: Node Exporter (host metrics) + prom-client (app metrics) + Prometheus + Loki + Promtail + Grafana — includes architecture flowgraph, all config files, systemd unit files, dashboard IDs, and EC2 security group port reference |
| `grafana-monitoring-stack` | `/grafana-monitoring-stack` | Battle-tested monitoring stack guide written for QA/DevOps engineers — all binaries installed in `/opt/monitoring`, includes pre-check (RAM/disk), port plan, phases 1–9, daily management commands, full troubleshooting section, and Grafana Cloud alternative |
| `nextjs-ubuntu-aws-deploy` | `/nextjs-ubuntu-aws-deploy` | Full Next.js deployment guide: Bitbucket SSH → clone → env → build → PM2 → Nginx → AWS ALB Target Group registration → SSL (Certbot) → Bitbucket Pipeline auto-deploy. Includes 502 troubleshooting checklist and quick reference card |
| `cicd_skill` | `/cicd_skill` | Complete CI/CD setup & debugging guide: Bitbucket Pipelines → S3 → AWS CodeDeploy → EC2 Ubuntu. Covers IAM setup, CodeDeploy agent, deployment scripts, Slack notifications, and 6 real failure lessons (bootstrap loop, npm ci lock file mismatch, NVM path issues, wrong Docker image, variable scoping, environment name mismatch) |

### Project-level (livguard-ecomm only)
| Skill | Usage | Description |
|-------|-------|-------------|
| `figma-livguard` | `/figma-livguard` | Livguard-specific Figma → code. File key pre-configured (`7AW0hKHcyl9yZUOvRYSA8j`), component map included, design tokens resolved, known quirks documented |

---

## 📚 Knowledge (non-project)
- [`tech-biography.md`](tech-biography.md) — technical biography; source doc for résumé / LinkedIn /
  GitHub / self-review.
- [`knowledge/`](knowledge/) — reusable playbooks & cross-project learnings (e.g. Fargate+API-Gateway
  deploy pattern, Bitbucket→CodeDeploy CI/CD, integration-service reference architecture).

## 🧱 Cross-cutting stack at a glance
- **Languages:** Java (8 → 25), Groovy/Grails, TypeScript/JavaScript, Kotlin (Gradle DSL).
- **Backend:** Grails 6/GORM, Spring Boot (1.5 → 4.0), Spring Cloud, JHipster, Node/Express.
- **Frontend:** React 18/19, Next.js 14/16, Remix, Vite SPAs, React Native/Expo, legacy Angular 4.
- **Data:** MongoDB, Snowflake (signature); PostgreSQL, Redis, Elasticsearch+Algolia+Atlas, SQLite.
- **Cloud/DevOps:** AWS `ap-south-1` (S3, SES/SNS, CodeDeploy+EC2, ECS Fargate+ECR, API GW/VPC Link/
  Cloud Map); Terraform; Docker; Bitbucket Pipelines + CircleCI; Sentry/Prometheus/Zipkin.
- **Domains:** Indian e-commerce/retail (payments, WMS, logistics, CRM) and fintech/NBFC KYC-credit.

---

*🌱 Free-form knowledge base. Add and update entries over time — projects are just one part.*
