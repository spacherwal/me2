---
name: feedback-use-skill-for-lsq-events
description: Always invoke the add-leadsquared-ecommweb-event skill before implementing any new LeadSquared EcommWeb activity
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ef4a7d2c-d78e-4362-81fa-1cda8a228d3f
---

Always invoke the `add-leadsquared-ecommweb-event` skill via the Skill tool BEFORE implementing any new LeadSquared EcommWeb activity. Do not start coding directly.

**Why:** User has had to remind multiple times. The skill exists specifically for this workflow and must be the starting point every time, no exceptions.

**How to apply:** Any time the user asks to add a new LeadSquared activity/event for the EcommWeb frontend, call `Skill({ skill: "add-leadsquared-ecommweb-event", args: "..." })` first, then follow the skill output to implement.
