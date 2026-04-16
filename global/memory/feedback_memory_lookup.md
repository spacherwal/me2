---
name: Always check global memory first
description: When recalling past work, always check both global and project-specific memory using Glob/Read tools, not Bash ls
type: feedback
---

Always check BOTH memory locations when the user asks to recall past work:
1. Global user-level: `C:\Users\SahilPacherwal\.claude\memory\`
2. Project-specific: `C:\Users\SahilPacherwal\.claude\projects\<project>\memory\`

**Why:** A Figma-to-React component for Livguard was saved in global memory but missed because only the project-specific path was checked. The user had to explicitly point to the correct path.

**How to apply:** On any "do you remember" or recall request, use Glob on both paths upfront. Use the Glob or Read tool directly — do NOT rely on Bash `ls` for Windows paths as it fails silently.
