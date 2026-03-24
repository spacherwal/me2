---
name: Coding and Communication Preferences
description: Sahil's preferences on how code and config should be written and how Claude should communicate
type: feedback
---

**No special/Unicode characters in config or properties files.**
Use plain ASCII comments only (e.g. `# Section name`, not `# ── Section ──────`).
**Why:** Sahil noticed and flagged the Unicode box-drawing characters immediately — they caused rendering concerns.
**How to apply:** In any `.properties`, `.yml`, `.env`, or similar config files, use only plain `#` comments with no decorative characters.

**Explain analysis before making changes.**
When reviewing code or config, present findings first and ask/confirm before executing.
**Why:** Sahil asks for analysis ("analyse this", "observe and suggest") before saying "proceed" or "yes". He wants to understand and agree with the approach first.
**How to apply:** On any review/refactor task, default to analysis-then-confirm flow unless the user explicitly says to just do it.

**Give honest, structured ratings and assessments.**
When asked to rate or review, provide a scored breakdown by category (security, quality, architecture, testing, etc.) with specific issues, not generic praise.
**Why:** Sahil explicitly asked for a rating and engaged positively with detailed, numbered findings including critical ones.
**How to apply:** Don't soften findings. List specific line numbers, class names, and concrete issues. Rate numerically.

**Prioritise security issues above all else.**
When security vulnerabilities are found, flag and fix them before anything else.
**Why:** Sahil immediately said "fix the credential exposure first" when multiple issues were identified.
**How to apply:** In any review, lead with security findings. When both security and quality issues exist, address security first.

**Do not use decorative section separators in any files.**
**Why:** Confirmed preference — Sahil asked to replace `──` style separators with plain comments across all properties files.
**How to apply:** All generated config/properties/yaml files should use simple `# Section Name` comments only.
