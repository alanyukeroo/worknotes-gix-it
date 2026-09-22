# ULS VLS Laser — Workstation Not Detecting Laser

**Date:** 2026-09-22
**Location:** GIX makerspace, ULS laser stations
**Scope:** ULS VLS4.60, 2x Windows 11 Pro 25H2 workstations (one is GIX-2025915)
**Status:** ✅ Resolved

---

## Summary

User ticket: lasers "not turning on," asked for the software on the included disc to be installed (no admin rights).

Lasers were powering on fine. The actual issue was one workstation not recognizing the laser over USB, caused by a leftover Windows registry flag that blocks all new device installs.

---

## Timeline

1. Installed UCP 5.38 from the disc on the first workstation. Laser detected as **ULS Print Support (VLS 0xB5)**, working.
2. Installed the same software on GIX-2025915. Laser showed as **Unknown device, Code 1**, HWID `USB\VID_10C3&PID_00B4`.
3. Manual driver update (Have Disk → ULS driver) failed with `0xe0000246`: "One of the installers for this device cannot perform the installation at this time."
4. `setupapi.dev.log` showed `ump: Device installation disabled` and repeated `Failed to control DeviceInstall service. Error = 0x00000425`.
5. Found `DeviceInstallDisabled = 1`, set to 0, laser detected. ✅

---

## Root Cause

```
HKLM\SYSTEM\CurrentControlSet\Services\DeviceInstall\Parameters
    DeviceInstallDisabled    REG_DWORD    0x1
```

Windows sets this during imaging / sysprep / major upgrades and clears it at the end. On GIX-2025915 it never got cleared. Already-installed devices (keyboard, mouse, monitor) keep working, so it only shows up when a new USB device needs a driver.

---

## Fix

PowerShell as admin:

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Services\DeviceInstall\Parameters" /v DeviceInstallDisabled /t REG_DWORD /d 0 /f
Restart-Service DeviceInstall -Force
pnputil /scan-devices
```

Then unplug/replug the laser USB. Device goes from Firmware Loader (`PID_00B4`) to **ULS Print Support (VLS 0xB5)** and UCP detects the laser.

---

## Ruled out along the way

| Check | Result |
|---|---|
| Driver packages in Driver Store | Present (ULS 5.38.57.99, `ulsprint.inf` as `oem0.inf`) |
| Pending reboot | None |
| Device Installation Restrictions GPO | Not set |
| `DeviceInstall` / `DsmSvc` services | Manual start, running |
| `HKLM\SYSTEM\Setup` (SystemSetupInProgress, SetupType, OOBEInProgress) | All `0x0` |
| Core isolation (Memory integrity, driver blocklist) | Not the cause |

---

## Gotchas

**VLS enumerates in two stages.** On plug-in the laser shows up as `ULS VLS360/460/660 Laser Engraver Firmware Loader` (`PID_00B4`). The PC pushes firmware, then it re-enumerates as `ULS Print Support (VLS 0xB5)`. If it's stuck as Unknown device with `PID_00B4`, the first stage failed.

**`sc` in PowerShell is `Set-Content`.** Use `sc.exe qc DeviceInstall`, otherwise it silently writes empty files named `qc` / `query` in the current folder.

**Slow restart + laser plugged in.** Before the fix, restarting with the laser USB connected hung for a long time, likely Windows retrying the blocked install. Should be gone now.

**COM2 is not the laser.** `Communications Port (COM2)` on GIX-2025915 is the motherboard serial port (`ACPI\PNP0501`).

---

## Open items

- Check other lab PCs from the same image for the flag:
  ```
  reg query "HKLM\SYSTEM\CurrentControlSet\Services\DeviceInstall\Parameters" /v DeviceInstallDisabled
  ```
  `0x1` = affected. Fix with the `reg add` above.
- To find when it started: first `Device installation disabled` entry in `C:\Windows\INF\setupapi.dev.log`, compare with imaging / Windows Update dates.
