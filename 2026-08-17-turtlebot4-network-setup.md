# TurtleBot4 Fleet — Network Setup & Debugging Notes

**Date:** 2026-08-17
**Location:** GIX B92 Robotics Lab
**Scope:** 4x TurtleBot4 (ROS_DOMAIN_ID 96, 97, 98, 99)

---

## Network Setup

**WiFi:** SSID `UW IOT`, password `<ask team lead>` (2.4GHz, 5GHz supported by Pi; Create3 base is 2.4GHz only)

**Subnet:** `10.155.32.0/24`, gateway `10.155.32.1`
- Static range: `.4`–`.154`
- DHCP pool: `.155`–`.254`
- **Convention:** static IP = ROS_DOMAIN_ID (e.g. domain 98 → `10.155.32.98`)

### Per-robot setup steps
1. Connect keyboard+monitor to Pi (HDMI0), login
2. `turtlebot4-setup` → Wi-Fi Setup → enter SSID/password → **Save** → **View Settings** to confirm → **Apply Settings** (easy to forget — nothing takes effect without this)
3. After reboot, get MAC: `ip link show wlan0`
4. networks.uw.edu (login `gixhelp`) → DNS Resources → `10.155.32.0/24` → open target IP → DHCP Static page → DHCP Options → hardware ethernet → Add → paste MAC
5. Wait ~5 min for propagation → `sudo reboot`
6. Verify: `ip addr show wlan0` shows the static IP

### Also verify on the Create3 side
Create3 has its own **ROS 2 Domain ID** setting (separate from the Pi's), accessible via its web UI at `192.168.186.2` (SSH tunnel from a laptop: `ssh -L 8080:192.168.186.2:80 ubuntu@<pi-static-ip>`, then browse `localhost:8080` → Application). This must match the Pi's domain ID or Create3 nodes won't be discovered.

---

## ROS2 Health Check Checklist (per robot)

```bash
ros2 node list                          # look for /motion_control, /robot_state, /_internal/*
ping -c 3 192.168.186.2                 # Pi <-> Create3 link over internal USB-C
sudo i2cdetect -y 3                     # Create3 base I2C bus
ros2 topic info /cmd_vel --verbose      # confirm /motion_control is subscribed
ros2 topic echo /wheel_status --qos-reliability best_effort --once
ros2 action send_goal /undock irobot_create_msgs/action/Undock "{}"
ros2 topic hz /scan
ros2 topic hz /oakd/right/image_raw/compressed   # or /oakd/rgb/preview/... — see below
ros2 action send_goal /dock irobot_create_msgs/action/Dock "{}"
```

**Gotcha — QoS mismatch:** Several Create3 topics (`/battery_state`, `/dock_status`, `/cmd_vel` subscriber side) publish with `BEST_EFFORT` reliability. Default `ros2 topic echo`/`pub` uses `RELIABLE`, which silently hangs waiting for a match — looks like "no data" but isn't. Always check `ros2 topic info <topic> --verbose` first; if `Reliability: BEST_EFFORT`, add `--qos-reliability best_effort` to echo/pub. Note `ros2 topic hz` doesn't support this flag — use `echo --once` instead to sanity-check those topics.

**Gotcha — Create3 nodes missing after Pi reboot:** Right after a Pi WiFi/network change, `ros2 node list` often shows only Pi-side nodes (turtlebot4_node, oakd, rplidar, etc.) with the Create3-side nodes (`/motion_control`, `/robot_state`, `/_internal/*`) absent, even though `ping 192.168.186.2` succeeds. Fix: restart the Create3 application via its web UI (`localhost:8080` → Application → **Restart application**, not the top-level "Reboot Robot" which can hang on "Requesting Reboot..."). Give it ~1 min; nodes reappear once the DDS session re-establishes.

---

## Per-robot findings

| Robot | Status | Notes |
|---|---|---|
| **96** | ✅ Working | No issues found |
| **97** | ✅ Working (after fix) | Initially: `i2cdetect -y 3` showed no responding devices, `turtlebot4_base_node` stuck in a connect retry loop to `/dev/i2c-3`, camera not publishing. **Root cause: loose cable / bad port** — reseating the cable and switching to a different port resolved it. |
| **98** | ✅ Working | First robot set up; used to validate the whole network + ROS2 flow. |
| **99** | ✅ Working (after fix) | Camera (OAK-D-Lite, MXID `184430102123591200`) not auto-loaded by the main bringup stack. Manually launching (`ros2 launch turtlebot4_bringup oakd.launch.py`) connected at `USB SPEED: SUPER` but then crashed repeatedly (`X_LINK_ERROR: Couldn't read data from stream 'sys_logger_queue'`, then `Device already closed or disconnected`). **Fix:** force USB2 mode before launching: `export DEPTHAI_USB2_MODE=1` then launch — camera connects at `USB SPEED: HIGH` and stays stable. **Also note:** this robot's camera topics are namespaced differently — `/oakd/rgb/preview/image_raw/compressed` instead of `/oakd/right/image_raw/compressed` used by 97/98. Check `ros2 topic list \| grep oakd` per robot rather than assuming the topic name. |

---

## Open items / things to watch
- Robot 99: camera requires the manual `DEPTHAI_USB2_MODE=1` launch step every time it's used — not persisted across reboots. Worth setting as a permanent env var or systemd override if this robot gets used often.
- General: worth reseating/checking all USB and I2C internal cabling on the other robots too, since 97's issue was purely a physical connection problem that looked like a software/driver issue for a long time.
