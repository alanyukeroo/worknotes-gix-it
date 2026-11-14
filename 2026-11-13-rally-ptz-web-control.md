# Logitech Rally — Browser PTZ Control and Setting the Camera Home Without the Remote

**Date:** 2026-11-13 (Friday shift)
**Location:** GIX conference room with a Logitech Rally system (Rally Camera `046d:0881`, Table Hub, audio)
**Scope:** Web app for pan/tilt/zoom and presets, macOS and Windows helpers, reverse-engineering the Rally's own Home/preset commands, IT staff docs
**Status:** 🟢 Working on macOS and in Zoom / Google Meet / Teams on Windows. The Windows helper is built and published but not yet tested on Windows hardware
**Time:** 1 shift. Commits show about 4.25 hours (11:46–16:03)
**Tools:** Worked with Claude Code. Vite + React + TypeScript, C (IOKit), C++ (Media Foundation, cross-compiled with mingw-w64), Vercel

---

## Summary

The room's Rally camera kept going back to its factory view (straight ahead, fully zoomed out) whenever someone plugged in their own laptop. The Rally **remote was missing**, so nobody could aim the camera or set a default position. Many different people use the room, so the fix had to work without anyone installing anything.

In one shift I:

- Built a web app ([rally-ptz-control.vercel.app](https://rally-ptz-control.vercel.app)) to pan, tilt and zoom the Rally from Chrome/Edge, with presets, keyboard shortcuts and a self-checking quick guide.
- Found why browsers on macOS can't move the Rally, and wrote a small macOS helper that talks to the camera over USB, with a one-line installer.
- **Reverse-engineered the Rally's own "Set Home" command** (what the remote does), so the room's default position is stored **in the camera** and works for everyone, on any laptop, without the remote.
- Built a Windows version of the helper and its PowerShell installer.
- Published the code to `UW-GIX/rally-ptz-control` and an IT staff guide on [uw-gix.github.io](https://uw-gix.github.io/it_staff/rally-ptz.html).

**Result:** the Rally now powers up aimed at the room's chosen position. Verified across USB unplug, power-adapter unplug, and on a Windows laptop with Zoom, Google Meet and Teams.

---

## Time Spent

| Block | Main work | Shown in commits |
| --- | --- | --- |
| Morning | Web app, macOS diagnosis, Mac helper, Vercel deploy, one-time Mac installer | 11:46–12:05 (plus build time before the first commit) |
| Midday | Position sync, room presets API, auto-recall on connect, XU reverse engineering and testing | 13:12–14:27 |
| Afternoon | "Make Rally Home" in the UI, quick guide, testing tools, Windows helper, docs | 15:18–16:03 |
| **Total** | | **About 4.25 h in commits** |

The commit figure is only what timestamps can prove. It doesn't cover hands-on testing with the camera, research, or the time before the first commit.

---

## Timeline

1. **Web app:** Vite + React + TypeScript using the W3C Camera PTZ API (`getUserMedia({ video: { pan: true, tilt: true, zoom: true } })` + `applyConstraints`). It uses the device's real ranges, throttles moves (latest value wins), and has a D-pad, sliders, presets (keys 1–6), and status messages for wrong browser, permission denied, camera busy and unplugged. Deployed to Vercel.
2. **macOS shows no PTZ:** Chrome on the Mac returned video but no `pan`/`tilt`/`zoom` at all in `getCapabilities()`. `ioreg` showed Apple's **`UVCAssistant`** holding the camera's video interfaces exclusively, with a `UVCCameraHideControls` rule in its configuration.
3. **Direct USB works:** a small IOKit program sending standard UVC control requests (`CT_PANTILT_ABSOLUTE`, `CT_ZOOM_ABSOLUTE`) on the default control pipe could read and move the camera, even while Chrome was streaming. Range: pan ±90°, tilt −90° to +50°, zoom 100–1500.
4. **Mac helper:** a single universal binary (C, IOKit + a tiny HTTP server on `127.0.0.1:47800`) that only answers `localhost` and the app's own site. Installed with one `curl … | bash` line as a LaunchAgent, with a checksum check. The web app detects it and switches over without reloading.
5. **Root cause of the "reset":** the Rally didn't reset on USB unplug. Its factory **Home** (where it goes on power-on) was the wide view, and the Windows Camera app sends its own zoom when it opens.
6. **Reverse engineering the Home command:** a read-only scan listed 8 Logitech extension units (~80 controls). Chromium's `webcam_private` source and the open-source `cameractrls` project pointed to the Logitech peripheral unit `{FFE52D21-8030-4E2C-82D9-F587D00540BD}`, selector 2. I tested each value on the camera, starting with the harmless ones (go to preset, then save preset, then set Home).
7. **Home stored in the camera:** set Home to the room's chosen preset, and the Rally beeped to confirm. It survived a power-adapter unplug and showed up correctly on a Windows laptop in Zoom, Meet and Teams.
8. **UI for it:** a 🏠 **Home** button on each preset (move there, wait until the motors stop, then save), plus **Go to Rally Home**.
9. **Staff experience:** an auto-ticking quick guide (Mac and Windows paths), a "Remove helper" option to retest first-run setup, and a camera-position sync so the UI follows moves made by other apps.
10. **Windows helper:** C++ using Media Foundation → `IKsControl` → the same Logitech unit, cross-compiled on macOS with mingw-w64. Installed per user with `irm … | iex` (no admin), with a sign-in auto-start.
11. **Docs and handover:** a clean export (desktop experiment excluded) pushed to `UW-GIX/rally-ptz-control` with an IT staff README, plus a guide page on the GIX IT site.

---

## Root Cause

Two separate problems looked like one:

1. **The camera's Home was the factory default.** The Rally moves to its Home on power-on. Normally you change it by holding Home on the remote for ~10 s, and the remote was gone.
2. **macOS hides the Rally's PTZ controls from browsers.** `UVCAssistant` owns the video interfaces, so Chrome on the Mac sees video only. UVC control requests on the device's default pipe still reach the camera.

---

## Fix: Logitech commands found

UVC `SET_CUR`, 1 byte, to the extension unit `{FFE52D21-8030-4E2C-82D9-F587D00540BD}` (unit 11 on this Rally, looked up by GUID), selector `2`:

| Byte | Meaning | Tested on the Rally |
| --- | --- | --- |
| `0x01` | Set Home (save current position) | ✅ beep, survives power cycle |
| `0x02` | Go Home | ✅ |
| `0x04`–`0x0B` | Save preset 1–8 in the camera | ✅ slot 1 |
| `0x0C`–`0x13` | Go to preset 1–8 | ✅ slot 1 |

For example, setting Home on macOS: `bmRequestType 0x21`, `bRequest 0x01`, `wValue 0x0200`, `wIndex 0x0B00`, data `01`.

Two open-source sources disagreed on `0x01`/`0x02` (Chromium: set/go Home; cameractrls: reset pan/tilt). Testing on the camera settled it: `0x01` didn't move the camera and saved the position, and `0x02` brought pan, tilt and zoom back. That matches Chromium.

---

## Ruled out along the way

| Check | Result |
| --- | --- |
| Browser permission ("use and move your camera") | Granted. Capabilities were missing entirely, not empty |
| Rally resets on USB unplug | No. Position kept when the laptop was undocked and redocked |
| Teams resets the camera | Not on most laptops. Only some AI laptops reframe (Windows Studio Effects) |
| Logi Sync / Logi Tune can set Home | Not documented anywhere I could find. Only the remote |
| Rally power from laptop USB | No. Camera, motors and hub run from the Rally's own adapter |

---

## Gotchas

**The Windows Camera app resets zoom.** It sends its own zoom when it opens. Home isn't erased, and the camera returns to it on the next power-on or "Go to Rally Home".

**Windows Studio Effects "Automatic framing"** on Copilot+/NPU laptops crops and follows faces in every app. It looks like the camera zooming but it's done on the laptop. Turn it off in Quick Settings → Studio Effects.

**RightSight** (Logi Tune/Sync auto-framing) makes the Rally ignore its saved Home while it's on.

**The Rally reads back about one step (1°) off** from the target position. Logitech documents ±2° tolerance for presets. The UI ignores changes smaller than that so sliders don't jitter.

**Vercel CDN right after a deploy** sometimes served the previous binary and checksum pair for a few seconds. Running the installer again picked up the new one.

**Probing localhost triggers Chrome's local network permission prompt.** On Windows the page only looks for the helper after someone clicks "Set up", so staff who only move the camera never see it.

---

## Open items

- Test the Windows helper on a Windows laptop (`rally-ptz-helper.exe probe`, then 🏠 Home).
- Optional: shared presets per room (`/api/presets`) need an Upstash Redis store connected in Vercel.
- The live site deploys from my personal repo, so changes have to be exported to `UW-GIX/rally-ptz-control` by hand. Point Vercel at the UW-GIX repo if it becomes the main one.
- Replace the missing Rally remote if budget allows (also gives staff the 2 hardware preset buttons).

---

## Links

- Live app: <https://rally-ptz-control.vercel.app>
- IT staff guide: <https://uw-gix.github.io/it_staff/rally-ptz.html>
- Code: `UW-GIX/rally-ptz-control` (private to UW-GIX)
