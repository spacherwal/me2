---
name: project_solarpro_integration
description: "SolarPro third LSQ account integration — scaffolding built 2026-06-23, placeholders to fill"
metadata: 
  node_type: memory
  type: project
  originSessionId: 560502bf-4a04-458e-9076-500548003c0e
---

Third independent LeadSquared account "SolarPro" (separate from default `lsq.*` and existing `solar.lsq.*`). Scaffolding built 2026-06-23 following the per-concern segregation pattern. Fetches agents + leads + 2 activities into new collections.

**Packages:** `model/solarpro`, `repository/solarpro`, `service/solarpro` (+`/impl`), `controller/solarpro`. Constants in `constant/SolarProLsqApiConstants` (reuses [[feedback_lsq_activity_pattern]]'s shared `LeadSquaredApiConstants` for headers/DATE_FORMAT/field keys).

**Config:** `solarpro.lsq.access-key/secret-key` (per-profile, env vars `SOLARPRO_LSQ_*` in dev/prod); 3 URLs `solarpro.lsq.{owners,recently.modified,activity}.url` in application.properties (default api-in21).

**Services** (interfaces keep SolarPro* names; account infra/config/package stays `solarpro`; data models + methods + endpoints + jobs are LG-prefixed): SolarProAgentService (`processLgSolarProAgents()` + agentNameCache + getAgentName/getLeadOwner), SolarProLeadService (`processLgSolarProLeads(from,to)`), SolarProActivityService (`processLgSolarProActivity(from,to)` + `processSolarProActivityTwo(from,to)` placeholder).

**Scheduler jobs** (ExternalScheduler dispatch + process methods): `lgSolarProAgents_job`, `lgSolarProLeads_job`, `lgSolarProActivity_job`, `solarProActivityTwo_job` (placeholder). ServiceLog names drop `_job`. scheduleConfig docs NOT yet inserted in Mongo.

**Agents DONE:** model renamed SolarProLsqAgent -> `LgSolarProLsqAgent` (collection `lgSolarProLsqAgent`), repo `LgSolarProLsqAgentRepository`, endpoint `/solarpro/lgSolarProAgents`, job `lgSolarProAgents_job` (ServiceLog `lgSolarProAgents`). Fields ID/FirstName/LastName/EmailAddress/Role/StatusCode/Tag/IsPhoneCallAgent (unchanged from LsqAgent mirror). NOTE: when renaming with Edit replace_all, `LgSolarProLsqAgent` CONTAINS `SolarProLsqAgent` — a global replace double-prefixes to `LgLg...`; rewrite the file instead.

**Backfill DONE** (segregated, mirrors [[project_lsq_backfill]]): `SolarProBackfillService(Impl)` in `service/solarpro`, progress model `SolarProBackfillProgress` (collection `solarProBackfillProgress`), repo `SolarProBackfillProgressRepository`. Endpoints: `GET /solarpro/backfill?fromDate&toDate&batchDays&activity`, `GET /solarpro/backfill/retry-failed?activity`, `POST /solarpro/backfill/ownerNames?activity`. Backfillable activity keys: `lgSolarProActivity`, `lgSolarProOutboundPhoneCallActivity`, `lgSolarProLeads` (agents excluded — full-list sync, no date range). Same 3-attempt 60/120/240s backoff, 5s inter-call delay, UTC batch gen, resume-safe skip-SUCCESS, background thread. ownerName backfill resolves via solarProAgentService.getAgentName (only works for leads' ownerId; activities store resolved email in owner + set ownerName at ingest, same quirk as default account).

**OPEN / TODO before production:**
- `application-prod.properties` edit was permission-denied — the 2 `solarpro.lsq.*=${SOLARPRO_LSQ_*}` lines must be added manually.
- Real SolarPro API host (datacenter may differ from api-in21) + env-var credential values.
- Activity ONE DONE: "LG Solar Pro" event 202 → model `LgSolarProActivity` (collection `lgSolarProActivity`), repo `LgSolarProActivityRepository`, service `processLgSolarProActivity(from,to)`, endpoint `/solarpro/lgSolarProActivity`, job `lgSolarProActivity_job` (ServiceLog `lgSolarProActivity`). Fields: mx_Custom_1=subActivityType, 2=l1Remarks, 3=l2Remarks, 4=s1FollowDate(Date). Plus standard ActivityEvent_Note/Status/Owner.
- Activity TWO DONE: "LG Solar Pro Outbound Phone Call" event 22 → model `LgSolarProOutboundPhoneCallActivity` (collection `lgSolarProOutboundPhoneCallActivity`), repo `LgSolarProOutboundPhoneCallActivityRepository`, service `processLgSolarProOutboundPhoneCallActivity(from,to)`, endpoint `/solarpro/lgSolarProOutboundPhoneCallActivity`, job `lgSolarProOutboundPhoneCallActivity_job` (ServiceLog `lgSolarProOutboundPhoneCallActivity`). Fields: mx_Custom_1=sourceNumber, 2=startCallTime(Date), 3=callDuration(Integer), 5=callOrigin, 7=rawCallStatus. Answered-only filter (skips Status != "Answered", same as default LSQ event-22); processedRecords still increments on skip so pagination terminates. Event 22 also = default LSQ OPC code but different account/service, no conflict. All placeholders now gone.
- Leads DONE: model renamed SolarProLsqLead -> `LgSolarProLsqLead` (collection `lgSolarProLsqLeads`), repo `LgSolarProLsqLeadRepository`, service `processLgSolarProLeads(from,to)`, endpoint `/solarpro/lgSolarProLeads`, job `lgSolarProLeads_job` (ServiceLog `lgSolarProLeads`). 86 fields mapped (82 from SOlarProLSQ_Leads.csv + 4 added 2026-07-03 via LTD-2807: dbStatus/mx_DB_Status, leadAllocationStatus/mx_Lead_Allocation_Status, asmName/mx_ASM_Name, rmName/mx_RM_Name — all String, added to both COLUMNS_CSV and mapLeadFields), plus ProspectID @Id and resolved ownerName; 11 Date fields parsed (LeadConversionDate, ModifiedOn, CreatedOn, mx_Site_Visit_Date, mx_Site_Visit_Completed, mx_Site_Rescheduled_Date, mx_Sale_Done_Date, mx_MQL_Date, mx_Follow_Up_Date, mx_Deal_Lost_Date, mx_Deal_Converted_Date), rest String.
- Insert 4 active scheduleConfig docs (cron, Asia/Kolkata).

Compiles clean with Corretto 17 (JDK at `~/.jdks/corretto-17.0.18`; Bash tool has no JAVA_HOME by default).
