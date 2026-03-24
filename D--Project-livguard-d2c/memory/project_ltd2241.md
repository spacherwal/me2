---
name: LTD-2241 WMS warehouse selection
description: Active branch dev-LTD-2241 implementing WMS warehouse selection in cart and checkout
type: project
---

Branch `dev-LTD-2241` adds WMS warehouse selection to the cart and checkout flow.

**Why:** Allow users/agents to select a specific warehouse (from `LocationWiseWarehousecode`) and have inventory checked against that warehouse via Vinculum.

**How to apply:** Changes span `LakshyaCartController`, `LakshyaCheckoutController`, `CheckoutService`, `LakshyaCartService`, `CartDTO`, `shippingPage.js`, `vueComponents.js`, and `_shippingPage.gsp`.
