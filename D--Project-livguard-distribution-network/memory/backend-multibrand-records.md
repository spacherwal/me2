---
name: backend-multibrand-records
description: "Backend redesign — retailers/distributors no longer phone-deduped; same physical retailer has one record per brand, each with a unique dealer_code"
metadata: 
  node_type: memory
  type: project
  originSessionId: 21ec4c7d-6ca2-455f-b4df-b18f5b36e5e2
---

As of the 2026-06 backend redesign, retailers and distributors are **no longer deduplicated by `PHONE_OFFICE`**. The same physical retailer can have **multiple DB records — one per brand it serves** (e.g. LIVGUARD + LIVFAST). `dealer_code` is unique per brand-record, so the two records share lat/lng + name but differ by `dealer_code` and `brand`, and each carries its own `TERRITORY_MAPPING` rows (per-brand distributor relationships).

**Why this matters:** the prior `DEDUP_QUALIFY` on `PHONE_OFFICE` was a data-loss bug — it deleted whole retailers (and their territory mappings) when distinct dealers shared a phone. Confirmed via local-vs-prod diff: Pune dropped 43 retailers / 198 active mappings, 95% unrecoverable from the surviving twin. The redesign fixes it.

**How to apply (frontend):**
- Co-located brand-records render as overlapping markers (same lat/lng, unique `dealer_code_lat_lng` id). Plan: group one pin per physical location, list per-brand counters in popup. Open question: the reliable "same physical retailer" grouping key now that phone is gone (coords may drift between brand-rows).
- Headline retailer counts now count brand-records, not shops. Decide primary unit (locations vs brand-counters).
- The `/api/retailers` `distributor_ids` join is correct per brand-record; the join was never the bug — only the dedup was.

Relates to [[global-search-plan]] (LTD-2581 map work).
