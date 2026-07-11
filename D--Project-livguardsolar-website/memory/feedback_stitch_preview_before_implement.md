---
name: feedback_stitch_preview_before_implement
description: Always show Stitch design screenshots to the user for approval before writing any implementation code
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a21a6b46-025c-4aaf-83a1-9f1f3b2299c5
---

After generating Stitch screens, download the screenshots locally, open them with `start ""`, and show them to the user via the Read tool. Wait for explicit approval before touching any code.

**Why:** User corrected this twice in the same session — jumped straight to implementation without showing the design. The preview step is the whole point of the Stitch-first workflow; skipping it removes the user's ability to redirect the design before code is written.

**How to apply:** In the executioner skill and any Stitch-driven task, the sequence is strictly: generate → download screenshots → show to user → get approval → implement. Never collapse generate and implement into one step.
