---
name: Ask Before Implementing
description: Present options and wait for user choice before writing code for architectural decisions
type: feedback
---

For architectural decisions, present the options with trade-offs and wait for the user to choose before writing any code.

**Why:** User prefers to understand the approach and make the call themselves rather than having code written speculatively.

**How to apply:** When there are multiple valid approaches (e.g. where to store state, how to structure an API call, context vs prop drilling), list the options with pros/cons and ask which to proceed with. Only write code after confirmation.
