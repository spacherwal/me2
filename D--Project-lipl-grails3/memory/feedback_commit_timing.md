---
name: feedback-commit-timing
description: Do not commit/push until user explicitly asks — always stop after editing and wait for test confirmation
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a7cef01b-c2e8-40e0-b652-85f09b61e353
---

Do not commit or push after making code edits unless the user explicitly asks to commit.

**Why:** User was mid-testing a fix and did not want the change committed before verifying it worked. Committing prematurely locked in an untested change.

**How to apply:** After editing files, stop. Report what was changed and wait for the user to say "commit" or "push". Never chain edit → commit → push in a single response unless the user explicitly asked for all three steps.
