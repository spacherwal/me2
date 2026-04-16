---
name: CartContext Architecture
description: CartContext owns all cart state — single API call shared across navbar count and cart page
type: project
---

CartContext (`src/context/CartContext.tsx`) is the single source of truth for cart data.

- Fetches cart via `getCartItems()` on mount, re-fetches when `isAuthenticated` changes (login → fetch, logout → clear in-memory)
- Exposes: `cartData`, `cartLoading`, `cartError`, `itemCount`, `addToCart`, `refreshCart`
- `itemCount` comes directly from `cartData.itemQty` — no local session tracking
- `addToCart` calls `fetchCart` after a successful add to keep count and summary in sync
- Cart page (`src/app/cart/page.tsx`) consumes from `useCart()` — no separate API call
- Auth handled via `useUser()` from `UserContext` — `UserProvider` wraps `CartProvider` in layout

**Why:** Avoids duplicate API calls when both navbar and cart page need cart data simultaneously.

**How to apply:** Any feature needing cart data should use `useCart()` — never call `getCartItems()` directly in a component.

**Known remaining issues (not yet fixed):**
- No AbortController — state update on unmounted component if navigating away mid-fetch
- Race condition if `addToCart` is clicked rapidly (multiple concurrent fetchCart calls)
- Navbar shows `Cart (0)` during initial load with no loading indicator
