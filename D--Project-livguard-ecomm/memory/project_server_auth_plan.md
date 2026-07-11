---
name: Server-Side Auth Token Plan
description: Pending implementation — auth token not available in server components because apiClient interceptor reads localStorage which is inaccessible on server
type: project
originSessionId: e2843612-fa33-4861-a246-64765e3f4f6b
---
## Problem
Server components call external APIs via `apiClient`. The interceptor reads the auth token from `localStorage` — unavailable on the server. The cookie fallback (`auth_token` via `next/headers`) is unreliable because `cookies()` is called inside an Axios interceptor async chain, risking AsyncLocalStorage context loss. Additionally, `apiClient` is a module-level singleton, unsafe for concurrent server requests.

**Current setup:**
- `UserContext.login()` saves token to `localStorage` AND sets `httpOnly` cookie `auth_token` via `POST /api/auth/set-token`
- `apiClient.ts` interceptor: client → localStorage, server → `cookies()` inside interceptor (flaky)

## Three Approaches Discussed (not yet implemented)

### Option 1 — `serverApiClient` factory (per-call axios instance)
- Create `lib/serverApiClient.ts` with `import 'server-only'`
- Factory function reads `cookies()` at invocation time (not inside interceptor)
- Returns a fresh pre-configured axios instance per call
- Server components import `serverApiClient` instead of `apiClient`
- Strip the server-side branch from `apiClient.ts` interceptor

### Option 2 — React `cache()` token resolver (RECOMMENDED)
- Create `lib/getAuthToken.ts` using React's `cache()` function
- `cache()` is request-scoped — deduplicates `cookies()` call across entire render tree
- No AsyncLocalStorage risk since it runs within React's render context
- API utility functions accept an optional token parameter
- Most idiomatic Next.js App Router pattern

### Option 3 — Middleware header injection
- `middleware.ts` reads `auth_token` cookie before request hits server components
- Injects it as `x-auth-token` request header
- Server components read via `headers()` from `next/headers`
- Works in Route Handlers and Server Actions too

## Files to Change (whichever option is chosen)
- `src/lib/apiClient.ts` — remove server-side interceptor branch
- `src/lib/getAuthToken.ts` — new file (Option 2)
- `src/middleware.ts` — new or update (Option 3)
- `src/app/products/page.tsx` — update to use server auth
- `src/app/products/[id]/page.tsx` — update to use server auth
- `src/app/orders/page.tsx` — update to use server auth

**Why:** Token is stored correctly (httpOnly cookie via `/api/auth/set-token`). Only the reading mechanism on the server side needs fixing.
