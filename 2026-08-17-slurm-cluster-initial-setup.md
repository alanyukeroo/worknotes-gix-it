# GIX GPU Cluster — Initial Slurm Setup

**Dates:** 2026-08-10, 08-11, 08-12, 08-17
**Location:** GIX GPU cluster controller
**Scope:** Slurm, Munge, GPU scheduling, networking, multi-node setup
**Status:** ✅ Slurm controller running. Later work continued in the [rollout notes](2026-10-30-gix-gpu-cluster-rollout.md)

> Reconstructed from the controller's admin logs, so it covers what was run and when. Some results weren't recorded.

---

## Summary

Set up Slurm on the GIX GPU controller, so GPU work could be scheduled instead of run by hand on individual machines. Then worked through networking, host names and controller startup problems to bring other machines into the cluster.

---

## Timeline

### 2026-08-10 — Slurm Install

1. Updated the system and installed build tools.
2. Installed Munge, set the key ownership and permissions (`munge:munge`, `400`), and enabled the service.
3. Installed `slurmctld`, `slurmd` and the Slurm client.
4. Created the spool folders and wrote `slurm.conf`, `gres.conf` and `cgroup.conf`.
5. Enabled and checked `slurmctld` and `slurmd`.
6. Installed and enabled SSH.
7. Started the static network configuration with netplan.

### 2026-08-11 — Network Configuration

Worked through several netplan changes and tightened the config file permissions to `600`.

### 2026-08-12 — Controller Setup Script

Ran and iterated on `setupcontroller.sh`, and updated `/etc/hosts` so the machines could find each other by name.

### 2026-08-17 — Getting the Controller Stable

1. Continued iterating on the setup script and host names, and flushed the DNS cache.
2. Set up admin permissions for cluster management.
3. `slurmctld` needed troubleshooting. Ran it in the foreground with `slurmctld -D -vvv` to see the error.
4. Corrected ownership of the spool folders.
5. Cleared stale state in `/var/spool/slurmctld`, then restarted `slurmctld` and `slurmd`.

---

## Skills

Slurm, Munge, GRES and cgroup configuration, netplan, Linux service troubleshooting, reading daemon debug logs.
