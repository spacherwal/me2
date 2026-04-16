---
name: Cart Empty State — 204 API Response
description: API returns 204/NO_CONTENT for empty cart, not 200 with empty array
type: project
---

The cart API returns `statuscode: 204` with `status: "NO_CONTENT"` and `data: null` when the user has no cart items — it does NOT return a 200 with an empty `cartItemList`.

**API response shape for empty cart:**
```json
{
  "data": null,
  "errors": [{ "message": "No Carts Available" }],
  "status": "NO_CONTENT",
  "statuscode": 204
}
```

**How to apply:** When handling the cart API response, always check for both `statuscode === 200` (has items) and `statuscode === 204 || status === "NO_CONTENT"` (empty cart). Treat 204 as a valid empty state, not an error.
