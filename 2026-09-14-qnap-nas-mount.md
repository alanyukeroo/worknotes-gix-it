# QNAP NAS — Shared Storage for the GPU Cluster

**Dates:** 2026-09-03, 2026-09-10, 2026-09-14
**Location:** GIX GPU cluster controller and QNAP NAS
**Scope:** Network routing, CIFS mount, mount at boot
**Status:** ✅ Mounted at boot. Credential handling still needs improvement

> Reconstructed from the controller's admin logs. The commands are recorded, but the reasons for each step are inferred.

---

## Summary

Connected the GPU cluster to the QNAP NAS at `/mnt/qnap`, so training data and results could be shared across machines. It's now the student project folder for cluster jobs.

---

## Timeline

### 2026-09-03 — Network Route

1. Installed `nfs-common`.
2. Removed a default route on `eno1` and checked connectivity to the NAS.

### 2026-09-10 — CIFS Mount

1. Created `/mnt/qnap` and tried mounting the `Public` share over CIFS.
2. Checked `dmesg` and installed `smbclient` to troubleshoot.
3. Mounted successfully after setting SMB `vers=3.0`.

### 2026-09-14 — Mount at Boot

1. Tried a root-only credentials file with an `fstab` entry.
2. That approach was backed out, and the share was mounted directly.
3. Added an `@reboot` entry with a short delay, so the mount waits for the network.

---

## Result and Follow-Up

The mount has come back by itself after later reboots, including the full cluster restart on 2026-10-28.

The boot entry still needs its credentials moved somewhere more secure. The earlier credentials file attempt is the place to start.

---

## Skills

CIFS/SMB, Linux routing, `fstab` and boot-time mounts, troubleshooting with `dmesg`.
