---
name: next-build-apk
description: "EAS APK build — DONE. First preview APK built 2026-07-05; project linked under Expo account spacherwal03"
metadata: 
  node_type: memory
  type: project
  originSessionId: 56738981-c761-4c2d-a5c6-89dc127614e6
---

**APK build: DONE (2026-07-05).** Setup completed this session:
- Repo initialized under git with the user's personal identity (Sahil Pacherwal / spacherwal03@gmail.com — see [[git-personal-account]]). No remote yet; user will push to https://github.com/spacherwal eventually.
- Expo login done (account `spacherwal03`). Project linked via `eas init --force` → `@spacherwal03/rent-tracker`, projectId `89618a1c-eca5-4c96-8680-45acd95559cc` (written into app.json + owner, committed).
- Cloud Android keystore generated (EAS stores it — no local keytool needed).
- First `preview` APK built successfully (buildType apk, sideloadable): https://expo.dev/accounts/spacherwal03/projects/rent-tracker/builds/d7b6148c-671d-4ab0-a153-c246dc91a585
- **Installed on the physical device via `adb install` and confirmed working as a standalone app.** `eas build:run` FAILED (looked for an emulator at `$ANDROID_HOME\emulator\emulator` — ANDROID_HOME is hacked to `C:\Users\SahilPacherwal` just so Expo finds adb). Reliable install of the cached build: `adb install -r (Get-ChildItem "$env:LOCALAPPDATA\Temp\eas-cli-nodejs\eas-build-run-cache" -Recurse -Filter *.apk | Sort LastWriteTime -Desc)[0].FullName`

**To rebuild:** `npx eas-cli build -p android --profile preview` (already logged in). Free tier = 15 builds/month — use `npx eas-cli update` for JS-only changes to avoid burning builds.

**On-device (2026-07-11):** FULL rent regression pass complete on both Expo Go and standalone APK — save, prefill, receipt share/print, history Share, reopen/edit, delete, dashboard refresh all confirmed. Phase 1 fully closed.

**Phase 2 kickoff decisions (2026-07-11, user-confirmed):** keep rate/electricity charge as-is; tenant/landlord names go in a Settings screen, both optional (needs a `settings` store, shared with editable-defaults item #6); off-device cloud backup IS wanted (Phase 3 item now in scope).

**Phase 2 progress (all 2026-07-11, typecheck+lint pass, NOT yet on-device validated):**
- #1 Expense categories DONE — `app/expense/new.tsx` + expense CRUD in `db/queries.ts`, dashboard tile live with this-month total.
- #6 Settings store DONE — `settings` + `categories` tables (schema.ts), `app/settings.tsx` (⚙ header button on dashboard) for tenant/landlord names, rent defaults (rent/rate/water, rent form prefills from these), and add/delete expense categories. Categories are now free-form (`Category = string`; `DEFAULT_CATEGORIES` replaced the old fixed union). Tenant/landlord now render on the receipt (`ReceiptParties`, "Billed to"/"From" block).
Next up: #2 history+filters, #3 dashboard rollups, #5 JSON export, then Phase 3 cloud backup.
