---
name: Stitch project setup — LivguardSolar360
description: Google Stitch MCP project and design system created for UI design generation in executioner skill
type: project
originSessionId: 77b1cd63-0762-4a53-a5b4-ca181b612884
---
Stitch project **LivguardSolar360** created (ID: `12280814496772576493`). The executioner skill uses this project to generate UI designs when a Linear task has no design reference.

**Why:** Figma is blocked (view-only seat, Starter plan quota). Stitch MCP is the replacement design generation path — generates high-fidelity screens from text prompts using the Livguard Solar brand tokens, then uses them as implementation source of truth.

**How to apply:** When the executioner skill runs Step 3b (no design reference), it checks for the "LivguardSolar Brand" design system in this project, creates it if absent, then calls `mcp__stitch__generate_screen_from_text` for both MOBILE and DESKTOP device types. Do not recreate the project — it persists across sessions.

Design system token summary:
- Primary: `#eb2a2b` (brand red)
- Neutral override: `#1f2022`
- Fonts: INTER (headline + body)
- Roundness: ROUND_FOUR
- Mode: LIGHT
- Variant: TONAL_SPOT
