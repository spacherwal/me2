---
name: feedback_placeholder_dash_not_skeleton
description: "Use dash — values for empty result cards, not skeleton bars"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a21a6b46-025c-4aaf-83a1-9f1f3b2299c5
---

For empty/pre-input state of a structured result card, show the real card layout with `—` dash values in muted gray rather than skeleton loading bars.

**Why:** Skeleton bars looked random and disconnected against labeled rows. Dash values mirror the real result structure so the user understands what they'll get, and the card looks intentional rather than broken.

**How to apply:** Whenever a result card has a "waiting for input" state, render the same component structure as the populated card but substitute every value with a `—` span styled in `secondary-300` (muted). Only use skeleton bars for async data that is genuinely loading.
