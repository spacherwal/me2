---
name: Ask before implementing solutions
description: User wants to see options before code is written for architectural/approach decisions
type: feedback
originSessionId: 9affc9fa-b3c7-431c-a63f-e772a6cf9843
---
Present solution options and wait for user confirmation before writing any code when there are multiple valid approaches to a problem.

**Why:** User rejected an implementation to ask for options first — they want to make the architectural decision themselves.

**How to apply:** When there are 2+ meaningful approaches to fix something (especially CORS, data fetching patterns, API architecture), list the options with trade-offs and wait for the user to pick one before writing any code.
