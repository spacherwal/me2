---
name: reuse-existing-fields
description: "Don't add redundant frontend fields when an existing one already carries the value"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3668fe73-999f-4d90-9c66-4965d9683968
---

When a value is already surfaced on a frontend type, reuse that field instead of adding a new one. Specifically: `phone_office` is already mapped to `Distributor.contact` and `MarkerRetailer.contactPhone` — use those for brand-grouping (Items 4–6), do NOT add a `phoneOffice` field.

**Why:** user rejected adding `phoneOffice` to types/routes when `contact`/`contactPhone` already held it. Prefers no duplicate fields.

**How to apply:** before adding a field to a type or route transform, grep for the underlying backend key (e.g. `phone_office`) to see if it's already mapped under another name; reuse it. Normalize in the consumer (these fields hold raw values, may be null/NA/blank).
