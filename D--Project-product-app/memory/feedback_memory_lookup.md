---
name: Always check global memory first
description: When recalling past work, always check global user-level memory at C:\Users\SahilPacherwal\.claude\memory\ in addition to project-specific memory
type: feedback
---

Always check BOTH memory locations when the user asks to recall past work:
1. Project-specific: `C:\Users\SahilPacherwal\.claude\projects\<project>\memory\`
2. Global user-level: `C:\Users\SahilPacherwal\.claude\memory\`

**Why:** A Figma-to-React component for Livguard was saved in global memory but missed because only the project-specific path was checked. The user had to explicitly point to the correct path.

**How to apply:** On any "do you remember" or recall request, use Glob on both paths upfront. Use `Glob("**/*.md", path="C:\\Users\\SahilPacherwal\\.claude\\memory\\")` — do NOT rely on Bash `ls` for Windows paths as it fails silently. Use the Glob or Read tool directly instead.
