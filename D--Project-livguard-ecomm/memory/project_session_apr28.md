---
name: Session Apr 28 2026
description: Razorpay card payment debugging + switch to Standard Checkout modal with method filtering
type: project
originSessionId: 2d07fc60-da89-4de2-94b7-3c582e6cf4d2
---
Switched Razorpay integration from custom checkout (`createPayment` / `razorpay.js`) to Standard Checkout modal (`rzp.open()` / `checkout.js`).

**Why:** `createPayment` triggers 3DS redirect (`"redirect": true` in Razorpay response) which fails without a `callback_url`. Standard modal handles 3DS internally.

**Key changes:**
- `checkout/layout.tsx`: `razorpay.js` → `checkout.js`
- `PaymentOptions.tsx`: removed inline card form entirely — card details entered in Razorpay modal
- `checkout/page.tsx`: removed CC_DC custom checkout branch; all methods (card, UPI, netbanking) now use `rzp.open()` with `config.display` filtered to selected method only
- `razorpay.ts`: removed `RazorpayCardDetails`, `RazorpayCreatePaymentOptions`, `createPayment` from interface

**Method filtering via `config.display`:**
- `show_default_blocks: false` + `instruments: [{ method: razorpayMethod }]`
- Bug found: API returns `'NET_BANKING'` (with underscore) not `'NETBANKING'` — fixed in `getRazorpayMethod` switch

**How to apply:** For any future Razorpay payment method additions, check the actual `navigationArea` value from the API response (log `selectedPaymentMethod.id`) before adding to the switch in `razorpay.ts`.
