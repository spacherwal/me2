---
name: project-assist-agent-nextjs
description: "Assist agent Next.js implementation plan — dual-token pattern, backend endpoints needed, frontend pages"
metadata: 
  node_type: memory
  type: project
  originSessionId: d7831c42-91ad-4941-b1b6-c529c899f942
---

Plan to implement assist agent in the Next.js D2C frontend using a dual-token pattern.

**Why:** Existing Grails implementation uses server sessions (`session['AssistAgent']`). Next.js D2C frontend is stateless (token-based auth via `/api/livdtc/**`), so sessions don't apply.

**Approach:** Dual-token — every request in assist mode sends `Authorization: customerToken` + `X-Assist-Agent-Token: agentToken`. `LivguardEcommInterceptor` validates the agent token and sets `request['assistAgentId']`.

**Key decisions:**
- Agent enters customer mobile number directly (no search UI)
- Backend creates `AuthenticationToken` for customer if one doesn't exist (`AuthenticationToken.findByUsername` or create)
- Both tokens stored in httpOnly cookies
- Only `LivguardEcommInterceptor` and order placement (`sale.assistAgent`) need backend changes

**New backend endpoints needed:**
- `POST /api/livdtc/assistAgent/login` — agent email+password → agentToken
- `POST /api/livdtc/assistAgent/impersonate` — agentToken + customerMobile → customerToken
- `POST /api/livdtc/assistAgent/sendOtp` — trigger OTP to customer mobile

**New domain:** `AssistAgentToken` (MongoDB) — tokenValue, agentId, dateCreated

**Full plan doc:** `PLAN-assist-agent-nextjs.md` in project root

**Status:** Plan drafted, pending senior review. Not yet implemented.

**How to apply:** When working on assist agent features, refer to this plan. The `AuthenticationToken` domain uses `username` field (not user reference), has no expiry.
