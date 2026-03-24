---
name: No pincode-based inventory fallback
description: User does not want pincode-based inventory lookup as a fallback when no wmsCode is selected
type: feedback
---

Do not add pincode-based inventory resolution as a fallback when no `wmsCode` is selected in cart/checkout flows.

**Why:** User explicitly did not ask for it. When no warehouse is selected, do nothing — skip inventory check entirely.

**How to apply:** In any WMS/Vinculum inventory logic, only check inventory when a `wmsCode` is explicitly provided. Never fall back to pincode-based `getInventory()` calls.
