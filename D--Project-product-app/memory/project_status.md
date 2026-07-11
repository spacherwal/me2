---
name: product-app current status
description: Current state of the product-app project — what's working, what was fixed, and what needs attention in production
type: project
---

App is running successfully on localhost:3000.

**Fixes applied:**
- Changed `VITE_API_BASE_URL` in `.env` from `https://uat.lockthedeal.com/api` to `/api` to route all API calls through Vite's dev proxy, resolving a CORS preflight rejection caused by the custom `version: 6005` header in `verifyOtp`.

**Known issue for production:**
The `version` header in `authService.ts` `verifyOtp()` is blocked by the server's CORS policy when called directly from the browser. The Vite proxy is a dev-only workaround. In production, either a backend proxy or an update to the server's `Access-Control-Allow-Headers` to include `version` will be needed.

**Why:** The backend (`uat.lockthedeal.com`) appears designed for native mobile app clients (which don't enforce CORS), not direct browser access.

**How to apply:** When resuming work, be aware that CORS + the `version` header is an unresolved production concern.
