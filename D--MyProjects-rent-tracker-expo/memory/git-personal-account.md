---
name: git-personal-account
description: "For this project use the user's PERSONAL GitHub account/identity for git, never the work one"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: db12789f-d143-4a4f-9e63-9c68efbb7f1e
---

For the rent-tracker-expo project, git must use the user's **personal** GitHub identity — NOT the work email `sahil.pacherwal@lakshyanet.com`.

Personal identity (set as this repo's LOCAL git config): **Sahil Pacherwal / spacherwal03@gmail.com**. Personal GitHub: https://github.com/spacherwal. Remote `origin` = https://github.com/spacherwal/rent-tracker-expo.git; `main` pushed & tracking as of 2026-07-05. Push auth resolves to the personal account via Git Credential Manager.

**Why:** This is a personal app; the user does not want it tied to their work identity/account, and will push it to their personal GitHub.

**How to apply:** Repo initialized with local `user.name`/`user.email` set to the personal identity above, so all commits here use it without touching global config. EAS APK build only needs a *local* git repo (or `EAS_NO_VCS=1`) — it does not require pushing to GitHub. See [[next-build-apk]].
