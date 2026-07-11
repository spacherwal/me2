---
name: No Auto Commits or Pushes
description: User wants code changes made locally only — no git commits and no git pushes unless explicitly asked
type: feedback
originSessionId: 84ab8ba4-68e6-468a-ac4d-38ee1018c690
---
Make all code changes locally but do NOT commit or push to git.

**Why:** User controls all git commits and pushes themselves.

**How to apply:** When executing tasks, write/edit files directly but skip any `git add`, `git commit`, or `git push` steps. Do not ask the user for commit approval — just skip it entirely. Only commit if the user explicitly says "commit this" or "push this".
