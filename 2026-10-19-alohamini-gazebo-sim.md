# AlohaMini2 — ROS 2 and Gazebo Simulation

**Dates:** 2026-10-13, last run 2026-10-19
**Location:** GIX GPU workstation
**Scope:** ROS 2, Gazebo, robot description, demo motion
**Status:** ✅ Demo ran to completion. A later sim log ended with a manual stop and a bridge crash

> Reconstructed from the files and logs on the machine.

---

## Summary

Built a ROS 2 and Gazebo package for the AlohaMini2 robot from its upstream repository, so the arms, lift and mobile base could be tried in simulation.

---

## What I Built

- **`make_pkg.py`:** generates the `alohamini2_urdf` ROS 2 package from the upstream repo, keeping the package name so the URDF's mesh paths still resolve.
- **`run_alohamini2_sim.sh`:** starts the robot in Gazebo, with RViz as an option.
- **`showcase.sh`:** a scripted demo that moves the camera, runs arm and lift motions, and drives the base over `/cmd_vel`.

---

## Result

- `showcase.log` reached `SHOWCASE_DONE`, with arm, lift and gripper targets logged along the way.
- `alohamini_sim.log` ends with `cmd_vel_to_base.py` stopped by a keyboard interrupt, and `parameter_bridge` exiting with code -11. The bridge crash wasn't investigated.

---

## Skills

ROS 2, Gazebo, URDF, Python, robot simulation.
