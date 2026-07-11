---
name: lsq-activities-implementation
description: "Full list of implemented LeadSquared activities, their codes, models, job keys, and the complete implementation checklist"
metadata: 
  node_type: memory
  type: project
  originSessionId: 336357cf-38c0-4416-8962-94e0d345b32c
---

## Implemented Activities

| Activity Name | Code | Model | Job Key |
|---|---|---|---|
| Razorpay Payment | 214 | LsqRazorpayActivity | lsqRazorpayActivity_job |
| Payment Link | 212 | LsqPaymentLinkActivity | (no scheduler job) |
| 360 All | 206 | LsqThreeSixtyAllActivity | lsqThreeSixtyAllActivity_job |
| Collect & Verify Document | 226 | LsqCollectVerifyDocumentActivity | lsqCollectVerifyDocumentActivity_job |
| Sale Verification | 218 | LsqSaleVerificationActivity | lsqSaleVerificationActivity_job |
| Loan Processing | 221 | LsqLoanProcessingActivity | lsqLoanProcessingActivity_job |
| Installation Process | 223 | LsqInstallationProcessActivity | lsqInstallationProcessActivity_job |
| Subsidy Processing | 225 | LsqSubsidyProcessingActivity | lsqSubsidyProcessingActivity_job |
| Comissioning Process | 224 | LsqComissioningProcessActivity | lsqComissioningProcessActivity_job |
| Delivery Activity | 222 | LsqDeliveryActivity | lsqDeliveryActivity_job |
| Outbound Phone Call | 22 | LsqOutboundPhoneCallActivity | lsqOutboundPhoneCallActivity_job |
| Refund Activity | 231 | LsqRefundActivity | lsqRefundActivity_job |

Refund (LTD-2782, 2026-06-30): mx_Custom_1=finalRemarks, mx_Custom_2=reasonForRefund. LG Solar360 = default LSQ account (default keys), not SolarPro.

## Checklist for Every New Activity

When adding a new LSQ activity, always touch all 7 of these:

1. **Model** — `model/leadsquared/LsqXxxActivity.java` (implements `Persistable<String>`)
2. **Repository** — `repository/LsqXxxRepository.java` (extends `MongoRepository` + `PagingAndSortingRepository`, has `findByProspectActivityId`)
3. **Constants** — add `ACTIVITY_XXX`, `CUSTOM_XXX_*` fields, `EVENT_NAME_XXX` to `LeadSquaredApiConstants`
4. **Service interface** — add `processLsqXxxActivity(Date fromDate, Date toDate)` to `LsqActivityService`
5. **Service impl** — inject repo, implement method in `LsqActivityServiceImpl` (includes `activityDate` + `createdByEmailId`)
6. **Controller** — add `GET /leadsquare/lsqXxxActivity` to `LeadsquareController`
7. **Scheduler** — add job case in `taskToBeExecuted` + private method in `ExternalScheduler`

**Backfill is a separate, closed registry** — not part of the 7-step checklist. To include an activity in backfill/retry, add it in 3 places in `LsqBackfillServiceImpl`: the `ACTIVITIES` list, the `OWNER_NAME_COLLECTIONS` map (`{collectionName, ownerField}`), and the `callActivity` switch.

## Standard Model Fields (every activity)

Base fields always present: `prospectActivityId` (@Id), `relatedProspectId`, `activityEventNote`, `activityEvent`, `activityEventName`, `activityDate`, `createdByEmailId`, `dateCreated` (@CreatedDate), `lastUpdated` (@LastModifiedDate)

Common optional fields: `status` (from `Status`), `owner` (from `Owner` via `lsqOwnerService.getLeadOwner()`)

## Scheduler Job Pattern

- Reads `lastSuccessRun` from `ServiceLog`; defaults to 48h lookback if null
- Calls `lsqActivityService.processLsqXxxActivity(fromDate, toDate)`
- Updates `ServiceLog` with "Success"/"Fail"

## MongoDB scheduleConfig Document

To activate a job insert into `scheduleConfig` collection:
```json
{
  "configKey": "lsqXxxActivity_job",
  "configValue": "0 10 8/6 * * *",
  "active": true,
  "dateCreated": ISODate(...),
  "lastUpdated": ISODate(...)
}
```
Existing jobs use cron `0 10 8/6 * * *` (runs at 08:10, 14:10, 20:10 IST).

## LsqAgent (Users.Get API)

- Endpoint: reuses `lsq.owners.url` = `Users.Get`
- Model: `LsqAgent` (collection: `lsqAgent`) — id, firstName, lastName, emailAddress, role, statusCode, tag, isPhoneCallAgent
- Service method: `processLsqAgents()` in `LsqOwnerService` / `LsqOwnerServiceImpl`
- Upsert pattern: `findById(id).orElse(new LsqAgent())` then save
- Controller: `GET /leadsquare/lsqAgents`
- Scheduler job key: `lsqAgents_job`
