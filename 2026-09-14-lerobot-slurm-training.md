# LeRobot Training on Slurm — First Tests

**Dates:** 2026-08-25, 2026-08-31, 2026-09-14
**Location:** GIX GPU cluster
**Scope:** LeRobot ACT training through Slurm, using a Python environment and a container
**Status:** ✅ Training produced checkpoints. Later replaced by the [Podman-based example](2026-10-30-gix-gpu-cluster-rollout.md)

> Reconstructed from files and job output on the controller.

---

## Summary

Tested whether a real robot-learning job could run through Slurm, using a dataset recorded with LeRobot. Both runs below produced a 2,000-step ACT checkpoint. These tests informed the student training workflow built later.

---

## Timeline

### 2026-08-25 — First Slurm Test

Wrote `test_slurm.py`, a small script for checking that a Slurm job runs.

### 2026-08-31 — LeRobot Job and Container Definition

1. Wrote `probe_node.sh` to inspect a node.
2. Wrote `slurm_test.sbatch`, which trains in a persistent Python environment.
3. Wrote `lerobot.def`, an Apptainer image based on `nvidia/cuda:12.8.1-cudnn-runtime-ubuntu24.04` with LeRobot installed, and `slurm_container.sbatch` to run it.
4. Trained ACT on the `record-test` dataset, with a checkpoint saved at step 2,000 (`act_record_test_14`).

### 2026-09-14 — Repeat Run

Moved the job to a dedicated `slurm-work` folder and trained again, with another checkpoint at step 2,000 (`act_record_test_16`).

---

## What Changed Later

- Apptainer and per-project Python environments were replaced by rootless Podman, because the GPU nodes don't have `pip` or `venv` set up.
- These early jobs were pinned to the controller. Training now runs on the 5 GPU nodes, and the controller runs no jobs.

---

## Skills

Slurm batch scripts, LeRobot, ACT, Apptainer, GPU containers, CUDA.
