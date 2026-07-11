# Memory Index

## User
- [user_profile.md](user_profile.md) — Who Sahil is, technical background, how he works and makes decisions

## LSQ Backfill
- [project_lsq_backfill.md](project_lsq_backfill.md) — Backfill system: files, batch logic, rate limit handling, endpoints, MongoDB monitoring

## ThreeSixtyLsqLead
- [project_three_sixty_lsq_lead.md](project_three_sixty_lsq_lead.md) — Lead/prospect model (not activity): wiring, field history, mx_Installation_Start_Date already exists, pmApplicationName added 2026-04-15

## SolarPro Integration
- [project_solarpro_integration.md](project_solarpro_integration.md) — Third LSQ account scaffolding (2026-06-23): packages, config, jobs, and placeholders/TODOs to fill before prod

## LSQ Activity Pattern
- [feedback_lsq_activity_pattern.md](feedback_lsq_activity_pattern.md) — Every new LSQ activity model must include activityDate + createdByEmailId; service must populate them before save
- [project_lsq_activities.md](project_lsq_activities.md) — Full list of implemented activities (9 total), 7-step checklist, scheduler pattern, scheduleConfig setup, LsqAgent details

## Feedback & Preferences
- [feedback_preferences.md](feedback_preferences.md) — No Unicode in config files, explain before executing, honest ratings, security first
- [feedback_compile_method.md](feedback_compile_method.md) — User compiles in IntelliJ IDEA, not Gradle CLI

## Project Status
- [project_optimization_status.md](project_optimization_status.md) — dev-optimization branch: what's done, pending tasks, and current rating (8.1/10)
- [project_springboot3_upgrade.md](project_springboot3_upgrade.md) — Spring Boot 3.5.9 upgrade: full change log, errors encountered, stashed pending server JDK 21 upgrade
