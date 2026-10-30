# ManiSkill — GPU Simulation Check on the RTX 5090

**Dates:** 2026-10-15, 2026-10-19
**Location:** GIX GPU workstation
**Scope:** ManiSkill3, GPU physics, Vulkan rendering, AI2-THOR bathroom scenes
**Status:** 🟡 Simulator verified. Scene loading has known issues

---

## Summary

Checked whether ManiSkill3 GPU simulation works on the RTX 5090, and whether AI2-THOR bathroom scenes could be used for a robot task (project "Balni"). Results are from the project's `RESULTS.md`.

---

## Benchmarks

| Benchmark | Speed | GPU memory |
| --- | --- | --- |
| `PickCube-v1`, 4,096 environments, state only | 244,960 environment steps/s | 2.65 GB |
| `PickCube-v1`, 256 environments, 128x128 RGB-D | 14,786 environment steps/s | 2.28 GB |

No PhysX GPU errors, and Vulkan rendering worked on the RTX 5090.

---

## Timeline

### 2026-10-15 — Environment and Benchmarks

1. Checked the OS, GPU, driver and Vulkan support.
2. Set up Python 3.12 with `uv`, since the system Python is 3.14, then installed PyTorch.
3. Installed ManiSkill 3.0.1 and SAPIEN 3.0.3, and ran the benchmarks.
4. Wrote `setup.sh` so the setup can be repeated.

### 2026-10-19 — AI2-THOR Bathrooms

1. The `AI2THOR` asset download died at 44% with no way to resume. Replaced it with `fetch_ai2thor.sh`, which can resume.
2. Confirmed that all 30 iTHOR bathrooms contain a toilet object, along with a sink and bathtub.
3. Loaded and rendered scenes in ManiSkill with `render_ithor.py`.

---

## Issues Found

- The AI2-THOR scene builder only has start poses for the Fetch robot in some scenes, and doesn't support other robots.
- `FloorPlan401` fails to render, because SAPIEN's GLTF loader rejects one of its door frame assets. Other scenes, such as `FloorPlan402`, are available to use instead.

---

## Skills

ManiSkill3, SAPIEN, GPU simulation, Vulkan, Python environments, performance benchmarking.
