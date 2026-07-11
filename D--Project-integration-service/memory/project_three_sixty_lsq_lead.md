---
name: ThreeSixtyLsqLead
description: ThreeSixtyLsqLead model structure, wiring, and field history — lead/prospect records from LSQ (not an activity)
type: project
originSessionId: b3d1675c-36d2-449b-b97b-1ca2ee2068ea
---
## Overview

`ThreeSixtyLsqLead` stores lead/prospect records from LeadSquared (not activity events).
Collection: `threeSixtyLsqLeads`

Distinct from `LsqThreeSixtyAllActivity` (activity code 206) — that tracks activity events; this tracks the lead record itself.

## Wiring

| Layer | File |
|---|---|
| Model | `model/leadsquared/ThreeSixtyLsqLead.java` |
| Repository | `repository/ThreeSixtyLsqLeadRepository.java` |
| Service interface | `service/LsqLeadService.java` — `processThreeSixtyLeads(Date fromDate, Date toDate)` |
| Service impl | `service/impl/LsqLeadServiceImpl.java` |
| Controller | `GET /leadsquare/threeSixtyLeads` in `LeadsquareController.java` |
| Scheduler | `ExternalScheduler.java` — job key: `threeSixtyLeads_job`, service log name: `threeSixtyLeads` |

## How LsqLeadServiceImpl Works

- Calls LSQ `Leads.RecentlyModified` API (POST, paginated, 500/page)
- `COLUMNS_CSV` constant lists all fields to fetch from LSQ
- `toFieldMap()` flattens `LeadPropertyList` into `Attribute -> Value` map
- `mapLeadFields()` maps flat fields onto the model using `get()` (String) and `parseDate()` (Date)
- Upsert: `findAllById(prospectIds)` for entire page, then `saveAll()`
- Rate limit guard: checks `MXAPIRateLimitExceededException` and breaks on hit

## Field History (additions by date)

| Field (model) | LSQ Attribute | Type | Added |
|---|---|---|---|
| `ownerName` | via `lsqOwnerService.getAgentName(ownerId)` | String | ~2026-04-03 |
| `deliveryRequestedDate` | `mx_Delivery_Requested` | Date | ~2026-04-03 |
| `installationRequestedDate` | `mx_Installation_Requested_Date` | Date | ~2026-04-03 |
| `pmApplicationName` | `mx_PM_Application_Name` | String | 2026-04-15 |
| `s2Agent` | `mx_Deal_Converted_By` | String | 2026-06-19 |

Note: Java field renamed `dealConvertedBy` -> `s2Agent` (commit 061ba86). LSQ attribute kept as `mx_Deal_Converted_By`. No `@Field` annotation, so Mongo key follows the Java name (`s2Agent`); pre-rename docs retain the old `dealConvertedBy` key.

## Note on mx_Installation_Start_Date

`installationStartDate` (`mx_Installation_Start_Date`) was already in the model, COLUMNS_CSV, and mapLeadFields before 2026-04-15 — do NOT add it again.
