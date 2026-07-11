---
name: me2 — Cross-project Memory & Knowledge Base
description: GitHub repo where Claude Code pushes learnings, memory, and cross-project knowledge
type: reference
---

All cross-session learnings and memory across Sahil's projects live in:
**https://github.com/spacherwal/me2** (clone at `C:/Users/SahilPacherwal/me2`).

**It is a free-form knowledge base, not just a project mirror.** Structure:
- `README.md` — the index/navigation layer: whole-estate table grouped by domain (Commerce,
  Distribution, Integration, Fintech/LivFin, Personal, Learning), stack + status + me2-presence per
  project.
- `tech-biography.md` — the engineer's technical biography (résumé / LinkedIn / self-review source).
- `knowledge/` — cross-project playbooks & reference architectures (e.g. Fargate+API-Gateway deploy
  pattern, Bitbucket→CodeDeploy CI/CD, integration-service reference architecture).
- `global/memory/` — user-level memory (applies across all projects).
- `D--<encoded-path>/memory/` — per-project curated memory. Folder names mirror
  `.claude/projects/<encoded>/`; **folders are never moved** (domain grouping is index-only) so the
  mirror never desyncs. Some folders also hold raw session `.jsonl` transcripts.

**Two-layer rule:** index/knowledge/tech-bio may hold derivable facts (stack/versions/architecture);
`memory/*.md` holds only non-obvious knowledge (decisions, constraints, gotchas, feedback).

**How to apply:** At the end of a significant session, refresh memory from the live sources —
`C:/Users/SahilPacherwal/.claude/memory/` → `global/memory/`, and
`.claude/projects/<enc>/memory/` → `D--<enc>/memory/` — then commit and push. When the user says
"me2", "our repo", or "our discussions repo", this is it.
