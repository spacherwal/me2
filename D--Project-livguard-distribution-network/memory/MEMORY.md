# Memory Index

- [Don't run build/restart commands](feedback_build_commands.md) — user runs `npm run build` / `npm start` in their IDE
- [Global search plan](global-search-plan.md) — deferred plan to make the search box work at Tier 1/2; details in repo docs/global-search-plan.md
- [Backend multi-brand records](backend-multibrand-records.md) — no more phone-dedup; same retailer = one record per brand, unique dealer_code; affects marker overlap + counts
- [Reuse existing fields](reuse-existing-fields.md) — don't add redundant fields; phone_office already lives in contact/contactPhone
