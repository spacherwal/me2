---
name: No Auto Commits
description: User does not want Claude to commit code automatically — user decides when to commit
type: feedback
originSessionId: 22aed296-943f-4abc-a40f-64da4047fe82
---
Never run `git commit` or `git add` + `git commit` automatically. Make file changes freely, but stop before any commit step and let the user decide when and what to commit.

**Why:** User explicitly stated they want to control all commits themselves.

**How to apply:** After completing file changes, present a summary of what changed and note that the user should commit when ready. Do not run `gsd-sdk query commit`, `git commit`, or any equivalent that creates a commit without explicit user instruction in that moment.
