# Robot Workstation — Provisioning on Ubuntu 26.04

**Date:** 2026-08-03
**Location:** GIX robotics workstation
**Scope:** Ubuntu 26.04, NVIDIA driver, CUDA, VS Code, Active Directory login
**Status:** ✅ Provisioning done. Final state wasn't recorded in the logs

> Reconstructed from the machine's admin logs and the script.

---

## Summary

Used `provision-robot-workstation.sh` to bring a fresh Ubuntu 26.04 machine up to a working robotics setup, so new machines don't have to be set up by hand. The script is safe to re-run, because each step checks whether it's already done.

**The script sets up:**
1. System updates and base tools
2. NVIDIA GPU driver
3. CUDA toolkit
4. Visual Studio Code
5. `sssd` and `realmd` for NetID logins through Active Directory, with an optional domain join

---

## Timeline

1. Downloaded and ran the provisioning script, with reboots in between.
2. Added a custom 1920x1080 display mode with `xrandr` on `HDMI-1`.
3. Downloaded the script again and re-ran it.
4. Adjusted `sssd.conf`, the PAM session settings and `sudo` configuration for AD logins.
5. Worked through NVIDIA driver options with `ubuntu-drivers`, `nvidia-driver-570-open` and `nvidia-driver-580-open`, ending on 580.

---

## Skills

Linux provisioning, Bash, NVIDIA drivers and CUDA, SSSD and Active Directory, PAM.
