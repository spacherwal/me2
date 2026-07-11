---
name: livguard-ecomm backend relationship
description: lipl-grails3 serves as the backend API for livguard-ecomm frontend project; GSD is active in both
type: project
originSessionId: cb4d4611-777e-40e0-bc71-004ecfa2457c
---
This Grails 3 project (`lipl-grails3`) is the **backend API** for a separate frontend ecommerce project called `livguard-ecomm`.

**Why:** livguard-ecomm is implementing UPI order placement and needed the exact backend API flow to integrate against.

**How to apply:** When working on API endpoints, controllers, or services in lipl-grails3, consider that livguard-ecomm is the consumer. Changes to payment flows, checkout APIs, or order endpoints may break the frontend integration.

## UPI Order Placement API endpoints (traced for livguard-ecomm)

| Step | Method | URL | Controller#Action |
|------|--------|-----|-------------------|
| 1. Place order | POST | `/lakshyaCheckout/commit` | `LakshyaCheckoutController#commit` |
| 2. Create Razorpay order | GET/POST | `/razorpay/submit.json?masterOrderId=&addressId=&amount=` | `RazorpayWebController#submit` |
| 3. Payment callback | POST | `/razorpayWeb/scan` | `RazorpayWebController#scan` |
| 4. Check payment status | GET | `/pay/check/{paymentId}` | `PayController#check` |
| 5. UPI success (Trupay/legacy) | POST | `/api/success/upi.json` | `LakshyaPaymentController#successResponseUPI` |

## Key response shapes
- `commit` success → forwards internally, frontend gets redirected to Razorpay flow
- `razorpayWeb/submit` → returns JSON: `{ razorpayorderid, amount, masterOrderId, paymentId, controller, action }`
- `pay/check` for UPI → returns JSON: `{ data: { redirectUrl, rsPaymentMethodUsed } }` (NOT a server redirect)
