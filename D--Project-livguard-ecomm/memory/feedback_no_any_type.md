---
name: No any type for fixes
description: User prefers proper typed solutions over using `any` to suppress TypeScript errors
type: feedback
originSessionId: eae3ee93-f1e0-4272-a368-117a20af72ea
---
When TypeScript build errors arise, do not use `any` as a workaround.

**Why:** User explicitly rejected `any` when fixing cart page type errors. Prefers proper typed interfaces.

**How to apply:** When a type mismatch occurs, fix it by extending the relevant interface with the missing fields as optional properties, or use proper generic constraints like `NonNullable<T>`. Only use `any` if the user explicitly asks for it.
