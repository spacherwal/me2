---
name: CartContext Architecture
description: CartContext owns all cart state — single API call shared across navbar count and cart page
type: project
originSessionId: eae3ee93-f1e0-4272-a368-117a20af72ea
---
CartContext (`src/context/CartContext.tsx`) is the single source of truth for cart data.

- Fetches cart via `getCartItems()` on mount, re-fetches on `isAuthenticated` change (login → fetch, logout → clear)
- Exposes `cartData`, `cartLoading`, `cartError`, `itemCount`, `addToCart`, `refreshCart`
- `itemCount` comes directly from `cartData.itemQty` (no local session tracking)
- `addToCart` calls `fetchCart` after a successful add to keep count in sync
- Cart page (`src/app/cart/page.tsx`) consumes from context — no separate API call

**Why:** Avoids duplicate API calls when both navbar and cart page needed cart data simultaneously.

**How to apply:** Any new feature needing cart data should use `useCart()` — never call `getCartItems()` directly in a component.
