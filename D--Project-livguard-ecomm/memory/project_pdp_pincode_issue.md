---
name: project-pdp-pincode-issue
description: "PDP shows \"Product not found\" for all products when default_pin_code cookie is missing — /api/web/offers.json requires x-pincode header"
metadata: 
  node_type: memory
  type: project
  originSessionId: 80555844-e557-48d5-adce-990026c7a2ab
---

`/api/web/offers.json` (called by `getProductDetails` in `src/lib/api/product.ts`) returns `status: BAD_REQUEST` with message "Please enter your 6 digit Pin Code." when no `x-pincode` header is present.

The axios interceptor in `apiClient.ts` only sets `x-pincode` if the `default_pin_code` cookie exists. First-time visitors without this cookie get an empty response body (`data: {}`), causing `apiResponse?.data?.offers?.[0]?.product` to be `undefined` → "Product not found" shown for every PDP.

**Why:** Backend requires a pincode to resolve stock/pricing for the offer. The `version: 100000` header (added commit `bceee6f`) may have tightened this validation.

**How to apply:** Backend team is fixing this server-side (2026-05-12). If the PDP issue resurfaces, the frontend fix is: resolve pincode (cookie → `/api/mobile/user.json` → `/api/web/defaultuserpincode.json`) before calling `getProductDetails`, then pass it explicitly via `x-pincode` header in the request config (third arg to `apiService.get`).
