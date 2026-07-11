---
name: No Build Check via CLI
description: User prefers to verify builds in their IDE, not via npm run build in terminal
type: feedback
originSessionId: ddeb20cd-465c-4434-ad36-b17799df3f1d
---
Don't run `npm run build` or other build checks via the terminal/Bash tool. The user checks compilation errors in their IDE.

**Why:** User preference — they will verify in the IDE.

**How to apply:** After making code changes, skip the build verification step. Just report what was changed and let the user verify.
