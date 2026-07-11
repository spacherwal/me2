---
name: feedback_no_git_checkout_uncommitted
description: Never git checkout tracked backend files — vernacularProvider and imageMetadataLibrary have uncommitted feature work
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a21a6b46-025c-4aaf-83a1-9f1f3b2299c5
---

Do not run `git checkout app/backend/vernacularProvider.server.tsx` or `git checkout app/backend/imageMetadataLibrary.server.tsx`. Both files carry uncommitted additions for in-progress pages (vernacular strings, page ID registrations) that are not committed to git and will be permanently lost on checkout.

**Why:** During a "start over" reset, git checkout wiped all uncommitted v3 vernacular strings and image metadata registrations, requiring full manual reconstruction from memory.

**How to apply:** To revert only the route file when restarting a task, restore it manually from context. If a full reset of the backend files is truly needed, check `git diff` first and save any additions elsewhere before checking out.
