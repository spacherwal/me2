---
name: Executioner skill run 3 — Figma design attempt (blocked)
description: Third fake task run (solar calculator). Figma path attempted via two accounts and generate_figma_design fallback. All blocked. Documented failure modes.
type: project
originSessionId: 469d1147-e2c8-48a2-a0c5-0b2d8bdb2f5d
---

## Task
Same fake task as runs 1 and 2: solar calculator page. This run tested the Figma design generation path.

## Attempt 1 — View-only Figma account
- Account: sahil.pacherwal@lakshyanet.com
- `whoami` returned `seatType: "view"` — Starter plan, files view-only
- `create_new_file` returned "Invalid planKey"
- Blocked immediately. Skill pre-flight gate worked correctly.

## Attempt 2 — New personal Figma account (spacherwal03@gmail.com)
- `whoami` returned `seatType: "expert"` — edit access confirmed
- New file created: key `CG8DwzyPXzbQX9D2W9I3xS`
- Library identified: Simple Design System
- Component keys retrieved: Button, Input Field, Tab, Accordion Item
- Variable keys retrieved: Background/Brand/Secondary, Background/Brand/Tertiary
- **Blocked at `use_figma` wrapper frame creation**: "You've reached the Figma MCP tool call limit on the Starter plan."
- ~6 `search_design_system` calls exhausted the MCP quota before any `use_figma` write could run.

## Attempt 3 — generate_figma_design fallback
Goal: bypass `use_figma` rate limit by using `generate_figma_design` (captures running web app).

- Injected `<script src="https://mcp.figma.com/mcp/html-to-design/capture.js">` into root.tsx — script didn't appear in rendered HTML (React/Remix SSR strips or ignores arbitrary `<script src>` in JSX head)
- Injected script dynamically via Playwright `browser_evaluate` — loaded successfully, `window.figma` confirmed ready
- `captureForDesign()` call hung indefinitely (5+ minutes) — DOM serialization on complex SSR page timed out or stalled
- **Blocked.** Total time spent on Run 3: ~40 minutes.
- root.tsx cleanup: reverted, no lasting changes to codebase.

## Key learnings
1. **`use_figma` and `search_design_system` share the same Starter plan MCP quota** — 6 search calls is enough to exhaust it, leaving nothing for write operations. Need to batch searches aggressively.
2. **`generate_figma_design` capture script injection via JSX `<script src>` doesn't work in Remix SSR** — React strips arbitrary script tags. Must inject dynamically via Playwright or add to `links()` export.
3. **`captureForDesign()` via Playwright `browser_evaluate` hangs on complex SSR pages** — the promise never resolves for large DOMs. Not a reliable fallback.
4. **Figma Starter plan is not viable for the executioner skill's Figma path** — both `use_figma` and `generate_figma_design` are blocked by quota/tooling limits.

## Skill pre-flight gate: WORKING
The `whoami` seat check added after run 3 correctly blocked the view-only account before wasting any MCP calls. That gate is validated.

## What's needed for a successful Figma run
- Figma Professional plan (or higher) to get sufficient MCP quota
- OR wait 24h for quota reset and batch all `search_design_system` calls into as few as possible (max 3-4 calls before writing)

**Why:** Track quality improvement across executioner skill runs and document real-world Figma MCP limitations.
