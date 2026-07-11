---
name: dev-run-usb
description: "How to run rent-tracker-expo on the physical Android phone — USB only, LAN/tunnel are blocked"
metadata: 
  node_type: memory
  type: project
  originSessionId: 56738981-c761-4c2d-a5c6-89dc127614e6
---

Running rent-tracker-expo in Expo Go MUST go over USB, not Wi-Fi. The dev machine is a corporate laptop: home Wi-Fi ("Airtel_SP") is classified **Public** (Windows Firewall blocks inbound 8081), the user has **no admin** (can't add a firewall rule or change the profile), and **ngrok tunnel is blocked** (`--tunnel` times out). So LAN and tunnel both fail with "Failed to download remote update".

**Working setup** (adb was downloaded standalone — no admin, no full Android SDK — to `C:\Users\SahilPacherwal\platform-tools\adb.exe`):

```powershell
$env:ANDROID_HOME="C:\Users\SahilPacherwal"          # parent of platform-tools, so Expo finds adb
$env:Path += ";C:\Users\SahilPacherwal\platform-tools"
adb reverse tcp:8081 tcp:8081                          # forwards phone loopback -> PC over USB
npx expo start --localhost
```
Then press `a`, or Expo Go -> Enter URL manually -> `exp://localhost:8081`. Phone needs USB debugging on and to be plugged in. Re-run `adb reverse` after any unplug or Metro restart. Env vars are per-terminal-session — re-set in a fresh terminal.

**Why `--localhost`:** pressing `a` without it opened the LAN URL (192.168.1.6, firewall-blocked). `--localhost` forces `exp://localhost:8081` which rides the adb reverse tunnel. See [[rent-tracker-sdk54]].
