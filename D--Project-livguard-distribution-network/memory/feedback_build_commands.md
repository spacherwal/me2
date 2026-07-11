---
name: feedback-build-commands
description: "Don't run build/restart commands — user runs those in their IDE"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a06f6c5b-eccb-4b99-853b-38ccbd23febb
---

Don't run `npm run build`, `npm start`, or server restart commands. The user prefers to run these themselves in their IDE.

**Why:** User explicitly said "I will run in IDE" when I attempted a production rebuild.

**How to apply:** When a fix requires a rebuild to take effect, describe the change and tell the user to rebuild — don't run it automatically.
