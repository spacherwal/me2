---
name: feedback-memory-scope
description: "Project-specific memories should be saved at project level only, not globally"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 80555844-e557-48d5-adce-990026c7a2ab
---

Save all memories related to this project (bugs, API quirks, architecture decisions, session notes) to the project-level memory path only: `C:\Users\SahilPacherwal\.claude\projects\D--Project-livguard-ecomm\memory\`.

**Why:** User wants project concerns scoped to the project, not polluting global or cross-project memory.

**How to apply:** Always write project/reference/feedback memories about this codebase to the project memory directory above. Do not create global-level memory entries for things that are specific to this project.
