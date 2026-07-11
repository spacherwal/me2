---
name: rent-tracker-sdk54
description: rent-tracker-expo is on Expo SDK 54; react-dom must stay pinned to 19.1.0 or npm install ERESOLVEs
metadata: 
  node_type: memory
  type: project
  originSessionId: 56738981-c761-4c2d-a5c6-89dc127614e6
---

rent-tracker-expo (D:\MyProjects\rent-tracker-expo) was upgraded from Expo SDK 53 to **SDK 54** on 2026-07-04: RN 0.81.5, React 19.1.0, expo-router ~6, expo-sqlite ~16. Verified via typecheck + lint + expo-doctor (18/18); NOT device-verified.

**Why:** `react-dom` is pulled in only as an optional peer by expo-router (web). If left unpinned, npm floats it to a newer version (e.g. 19.2.7) that demands react@^19.2.7, conflicting with SDK 54's react@19.1.0 → `npm install` fails with ERESOLVE.

**How to apply:** keep `"react-dom": "19.1.0"` pinned in package.json to match `react`. If re-installing, prefer a clean `npm install` over `expo install --fix` (the latter's internal install can hit a transient ERESOLVE). The 4 remaining deprecation warnings (glob@7, inflight, rimraf@3, uuid@7) are RN build-toolchain deps — unfixable at app level, do NOT use `audit fix --force` or npm overrides.
