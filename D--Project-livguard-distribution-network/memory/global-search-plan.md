---
name: global-search-plan
description: "Future plan to make the left-sidebar search box work at Tier 1/2, not just Tier 3"
metadata: 
  node_type: memory
  type: project
  originSessionId: 3608f638-27eb-463c-b11c-68766b8e60dc
---

The left-sidebar search box currently only filters loaded Tier-3 markers (writes `filters.search`, consumed in `leaflet-map.tsx`); it's dead at all-India/state level and is now gated to render only at Tier 3.

A phased plan to make it a global search/locator is written up in the repo at `docs/global-search-plan.md`. Phases: (1) states-only client-side locator, (2) add district index, (3) backend `/search` endpoint for distributors + retailers as a grouped typeahead. Key constraint: don't eagerly load all 13.7k retailers client-side — that breaks the tiered-loading perf design. Open decision gating phase 3: whether a backend `/search` endpoint is on the table.

Deferred by the user on 2026-06-22 ("will implement later").
