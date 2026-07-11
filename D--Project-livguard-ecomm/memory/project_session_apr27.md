---
name: Session Apr 27 2026 — Razorpay Payment Integration
description: Phases 2, 3 (removed), and card payment (CC_DC) implemented on feature/razorpay-integration
type: project
originSessionId: cb8a9374-bcde-4953-b5e4-ce15307fa544
---

Completed Phase 2, attempted Phase 3 (removed), and implemented inline card payment on branch `feature/razorpay-integration`.

**Why:** User wanted checkout payment method selection, UPI verification (later removed — deprecated), and inline card form using Razorpay custom checkout.

**How to apply:** Reference when continuing Razorpay work or adding netbanking/EMI phases.

## Phase 2: Razorpay Payment Method Pre-Selection (complete, committed)

Added `config.display.blocks` to Razorpay options so the modal shows only the selected method.

Key files:
- `src/lib/razorpay.ts` — `RazorpayMethod` type, `RazorpayDisplayConfig`, `getRazorpayMethod()`, `getRazorpayMethodDisplayName()`
- `src/app/checkout/page.tsx` — `razorpayConfig` wired into Razorpay options

## Phase 3: UPI ID Collection — REMOVED (UPI Collect deprecated Feb 2026)

Was implemented then removed. Razorpay's `/v1/payments/validate/vpa` endpoint returns "The requested URL was not found on the server" because UPI Collect was deprecated on 28 Feb 2026. UPI now works via Intent/QR in the Razorpay modal. No inline UPI form needed.

## Card Payment: Inline form via `rzp.createPayment()` (implemented locally, not committed)

Custom checkout approach — card data goes browser → Razorpay SDK directly, never through our server (SAQ-A-EP PCI compliant).

**Critical: The card option's `navigationArea` from the Livguard API is `'CC_DC'`, NOT `'CREDIT_DEBIT'`.**

Key files modified:
- `src/lib/razorpay.ts` — added `RazorpayInitOptions`, `RazorpayCardDetails`, `RazorpayCreatePaymentOptions`; extended `RazorpayInstance` with `createPayment()`, `payment.success`, `payment.error`; `Window.Razorpay` accepts `RazorpayOptions | RazorpayInitOptions`; `getRazorpayMethod` maps `CC_DC → card`
- `src/components/checkout/PaymentOptions.tsx` — `onCardDetailsChange` prop, card state, `isCardFormValid` memo, `useEffect` propagation, card form UI (number/name/expiry/CVV) shown when `CC_DC` selected, Place Order gated on `isCardFormValid`
- `src/app/checkout/page.tsx` — `cardDetails` state, `handleCardDetailsChange` useCallback, `CC_DC` branch in `handleProceedToPayment` using `rzp.createPayment()` with `payment.success`/`payment.error` handlers

Skill saved at: `C:\Users\SahilPacherwal\.claude\skills\razorpay-card-payment-skill\SKILL.md`

## Env vars required

`.env.local`:
- `NEXT_PUBLIC_RAZORPAY_KEY_ID=rzp_test_xxx` (client-side, for both modal and createPayment)
- `RAZORPAY_KEY_SECRET` — no longer needed (VPA proxy deleted)

## Deferred phases

- Netbanking bank selection — not started
- COD/CARDONDELIVERY — unchanged
