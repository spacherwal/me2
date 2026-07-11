---
name: project-status
description: Current working state of the livguard-integration project
metadata: 
  node_type: memory
  type: project
  originSessionId: 2012a5ec-4bf5-4f4a-b028-5e64fa866a9d
---

End-to-end flow is working: third-party POSTs to `/api/retailers` → Spring Boot → Snowflake MERGE INTO → data confirmed visible in Snowflake.

All three entities implemented: Retailer, Distributor, TerritoryMapping.

**Retailer and Distributor models rebuilt (2026-06-03) to match exact third-party JSON field names.** Both models are now flat with snake_case-aligned camelCase fields (Jackson SNAKE_CASE strategy serializes them correctly). Retailer PK: `customer_number_c`. Distributor PK: `phone_office` (no ID field in source data). All address fields use `billing_address_*` naming. Retailer includes geo (`latitude_c`, `longitude_c`) and potential (`ib_potential_c`, `fourw_potential_c`, `ups_potential_c`) fields. Distributor has no geo/potential/date fields — simpler schema. Validated working with real curl tests against Snowflake.

**API Key security is live.** `X-API-Key` header required on all `/api/**` endpoints. `/actuator/health` and `/actuator/info` are public. Keys configured via `api.security.keys` in profile yml. Local defaults: `local-dev-key-1`, `local-dev-key-2`. Prod/dev read from `API_KEY_1`, `API_KEY_2` env vars.

**Auth endpoint live.** `POST /api/auth/validate` — public endpoint (no X-API-Key), accepts `{email, password}`, returns `{id, email, name}` on 200 or empty 401. Uses `BCryptPasswordEncoder`. `PUBLIC.USERS` table in Snowflake. `AdminSeeder` seeds `admin@livguard.com` / `password123` on first startup.

**GeoLocation master table implemented.** `PUBLIC.GEO_LOCATION` stores Indian city/pincode/lat-long data. Fields: `id`, `countryCode`, `postalCode`, `placeName`, `state`, `stateCode`, `district`, `districtCode`, `subDistrict`, `subDistrictCode`, `latitude`, `longitude`, `accuracy`, `source`, `createdAt`, `updatedAt`. Seeded from GeoNames IN_dedup.txt (19,238 unique pincodes deduped from 155,570 rows). File at: `C:\Users\SahilPacherwal\OneDrive - Lakshya Internet Private Limited\Pictures\Screenshots\IN\IN_dedup.txt`. Seed endpoint: `POST /api/geo-locations/seed/file` (multipart upload). All geo reads served from in-memory `GeoLocationCache` (loaded at startup) — zero Snowflake queries at runtime. Nominatim fallback for unknown locations persists + warms cache. Resolve order: postalCode → placeName+state → district+state → Nominatim.

**Why:** LTD-2581 — ingest third-party entity data into Snowflake.

**How to apply:** Core plumbing is done. Pending: TerritoryMapping alignment to source data, frontend endpoints, deployment.
