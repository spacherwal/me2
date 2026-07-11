---
name: Razorpay Standard SDK Context
description: Complete context for Razorpay Standard Checkout integration — current state, decisions, known issues, and what's left
type: project
originSessionId: 2d07fc60-da89-4de2-94b7-3c582e6cf4d2
---

## Current State (as of Apr 29 2026)

Razorpay Standard Checkout is fully wired up and committed. COD, NEFT, CHEQUE handled server-side; card, UPI, netbanking open the Razorpay modal.

**Script loaded:** `checkout.js` (strategy="afterInteractive")
**Location:** `src/app/checkout/layout.tsx`

---

## Architecture

### Flow
1. User selects payment method on checkout page
2. User clicks "Place Order"
3. `createOrder` API called with `paymenttype = selectedPaymentMethod.id` → returns `grandTotal` + `razorpayorder.id`
4. NEFT/CHEQUE/COD → `PAYMENT_HANDLERS[paymentType]` → redirect to `/order-success`
5. All other methods → `new window.Razorpay(options); rzp.open()`
6. On `handler` (success) → `updatePaymentStatus` → redirect to `/order-success`
7. On `payment.failed` → toast error, `setIsProcessingPayment(false)`
8. On modal dismiss → toast warning, `setIsProcessingPayment(false)`
9. `modalOpened` flag in finally block prevents premature spinner reset

### Method Filtering (how only selected method shows in modal)
`config.display` passed to Razorpay options:
```js
config: {
  display: {
    blocks: { primary: { name: "...", instruments: [{ method: razorpayMethod }] } },
    sequence: ["block.primary"],
    preferences: { show_default_blocks: false }
  }
}
```
Built via `getRazorpayMethod(selectedPaymentMethod.id)` — returns `undefined` for unknown methods → no `config` key → Razorpay shows all methods.

### navigationArea → Razorpay method mapping (`src/lib/razorpay.ts`)
```
UPI          → 'upi'
CC_DC        → 'card'
NETBANKING   → 'netbanking'
NET_BANKING  → 'netbanking'   ← actual API value (with underscore)
NEFTRTGS     → 'netbanking'
CHEQUE       → 'netbanking'
(unmapped)   → undefined → no config → Razorpay shows all methods
```

### PAYMENT_HANDLERS (server-side, no modal)
```typescript
const PAYMENT_HANDLERS = {
  NEFT:   (orderId) => payViaNEFT({ masterOrderId: orderId }),
  CHEQUE: (orderId) => payViaCHEQUE({ masterOrderId: orderId }),
  COD:    (orderId) => payViaCOD({ masterOrderId: orderId }),
};
```
Matched via `selectedPaymentMethod.id?.toUpperCase()`.

### razorpayOrderId extraction
```typescript
const razorpayOrderId = response?.razorpayorder?.id || response?.razorpayOrderId;
```

---

## Key Files
| File | Role |
|---|---|
| `src/app/checkout/layout.tsx` | Loads `checkout.js` script (afterInteractive) |
| `src/lib/razorpay.ts` | Types + `getRazorpayMethod` + `getRazorpayMethodDisplayName` + Window declaration |
| `src/app/checkout/page.tsx` | `PAYMENT_HANDLERS` + `handleProceedToPayment` — full payment orchestration |
| `src/components/checkout/PaymentOptions.tsx` | Payment method selector UI (radio buttons, COD pre-selected) |
| `src/lib/api/checkout.ts` | `createOrder`, `payViaCOD`, `payViaNEFT`, `payViaCHEQUE`, `updatePaymentStatus` |

---

## Decisions Made
- **Standard Checkout modal** (not custom/headless) — avoids 3DS callback_url complexity
- **No inline card form** — card details entered inside Razorpay modal
- **No `createRazorpayCustomer` call** — only needed for saved cards (not implemented)
- **Surcharge/discount** — server-side: `createOrder` returns correct `grandTotal` per method
- **`razorpayorder.id`** is the Razorpay order ID field from the API response (not `razorpayOrderId`)

---

## Status
Razorpay integration is complete and committed. Do not treat any part of this as pending.
