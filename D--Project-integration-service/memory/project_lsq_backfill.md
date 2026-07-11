---
name: LSQ Backfill Feature
description: Full context of the LSQ activity backfill system — files, logic, rate limit handling, endpoints, and MongoDB monitoring
type: project
---

## Files

| File | Purpose |
|---|---|
| `model/leadsquared/BackfillProgress.java` | MongoDB doc (collection: `lsqBackfillProgress`) — fields: activity, batchFrom, batchTo, status (PENDING/SUCCESS/FAILED), attempts, lastAttemptAt |
| `repository/BackfillProgressRepository.java` | findByActivityAndBatchFromAndBatchTo, findByStatus, findByActivityAndStatus |
| `service/LsqBackfillService.java` | Interface: startBackfill(fromDate, toDate, batchDays, activity), retryFailed(activity) |
| `service/impl/LsqBackfillServiceImpl.java` | Core logic: batch generation, retry with backoff, progress tracking, background thread |
| `controller/LeadsquareController.java` | GET /leadsquare/backfill and GET /leadsquare/backfill/retry-failed |

## How It Works

- Splits date range into batchDays-sized chunks using **UTC Calendar**
- batchEnd is set to **23:59:59.999 UTC** to avoid gaps between batches
- Next batchStart = batchEnd + 1ms, reset to midnight — no overlap, no gap
- Processes all 7 activities (or a specific one if `activity` param passed) per batch sequentially
- Skips batches already marked SUCCESS (resume-safe on restart)
- **5s delay** between every API call (proactive rate limit avoidance)
- Retry backoff: **60s -> 120s -> 240s** (up to 3 attempts) when service returns false
- Runs in a **background thread** — HTTP returns immediately

## Rate Limit Handling (in LsqActivityServiceImpl)

LSQ returns this on rate limit (not an exception — a normal HTTP 200 response):
```json
{"Status":"Error","ExceptionType":"MXAPIRateLimitExceededException","ExceptionMessage":"API calls exceeded the limit of 20 in 5 second(s)"}
```

All 9 `processLsqXxxActivity()` methods check after getting responseBody:
```java
if (responseBody != null
        && "Error".equals(responseBody.get("Status"))
        && "MXAPIRateLimitExceededException".equals(responseBody.get("ExceptionType"))) {
    success = false;
    break;  // exits the while loop, returns false to backfill service
}
```
The `break` is critical — without it the while loop hammers LSQ infinitely on rate limit.

## Activities in Backfill (7 total)

- lsqThreeSixtyAllActivity
- lsqCollectVerifyDocumentActivity
- lsqSaleVerificationActivity
- lsqLoanProcessingActivity
- lsqInstallationProcessActivity
- lsqSubsidyProcessingActivity
- lsqComissioningProcessActivity

Note: lsqRazorpayActivity and lsqPaymentLinkActivity are NOT in the backfill list.

## Endpoints

```bash
# All activities
GET /leadsquare/backfill?fromDate=2026-01-01&toDate=2026-03-31&batchDays=3

# Single activity
GET /leadsquare/backfill?fromDate=2026-01-01&toDate=2026-03-31&batchDays=3&activity=lsqThreeSixtyAllActivity

# Retry all failed
GET /leadsquare/backfill/retry-failed

# Retry single activity failed
GET /leadsquare/backfill/retry-failed?activity=lsqThreeSixtyAllActivity
```

## Monitor Progress in MongoDB

```js
db.lsqBackfillProgress.countDocuments({status: "SUCCESS"})
db.lsqBackfillProgress.countDocuments({status: "FAILED"})
db.lsqBackfillProgress.countDocuments({status: "PENDING"})
```

Multiple API logger entries with the same date range but different PageIndex values are **normal** — it is internal pagination (500 records/page), not batch repetition.
