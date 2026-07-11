---
name: Executioner skill run 2 — solar calculator page
description: Second fake task run using the executioner skill (solar calculator page). All 5 run-1 issues fixed. 3 new minor issues found. Self-rated 8.5/10.
type: project
originSessionId: 469d1147-e2c8-48a2-a0c5-0b2d8bdb2f5d
---
## Task
Same fake task as run 1: solar calculator page at `/solar-calculator` with bill/units input, result card, CTA.

## Run 1 issues — all fixed
1. `result!` non-null assertions → `ResultCard` prop typed as non-nullable; parent conditionally renders
2. `type="number"` → `type="text" inputMode="numeric" pattern="[0-9]*"` with onChange digit filter
3. Animation not replaying → `key={calcCount}` counter increments on every Calculate click
4. File reads too late → verified backend registrations before writing
5. Interactive state not verified → clicked Calculate and confirmed result card on desktop + mobile

## New issues found in run 2 (rated 8.5/10)
1. `onChange` strips decimal point — users can't enter fractional kWh (e.g. 375.5). Should allow `[0-9.]*` and validate decimal format.
2. Desktop placeholder text is a hardcoded English string, not from `ContentProviderContext` — breaks vernacular/Hindi support.
3. `parseFloat` on a digit-only string — should be `parseInt(inputValue, 10)` for clarity since decimals are stripped.

## Skill fixes to apply after run 2
- Step 5: Add rule — any user-visible string, including placeholders and empty states, must use `ContentProviderContext`; never hardcode English text in JSX.
- Step 5: Add rule — when stripping non-numeric input, decide explicitly whether decimals are needed and use `[0-9]*` vs `[0-9.]*` accordingly.

## Screenshots (on disk)
- `D:\Project\livguardsolar-website\solar-calc-v2-desktop.png` — desktop 1280px, initial state
- `D:\Project\livguardsolar-website\solar-calc-v2-desktop-result.png` — desktop, result card visible (₹28,800, 2.5kW, 3,750kg)
- `D:\Project\livguardsolar-website\solar-calc-v2-mobile-result.png` — mobile 375px, result card visible (₹48,000, 4.2kW, 6,250kg)

**Why:** Track quality improvement across executioner skill runs. Run 1 was 7/10, run 2 is 8.5/10. The gap closed by fixing all 5 known issues; 3 new (smaller) issues surfaced.
