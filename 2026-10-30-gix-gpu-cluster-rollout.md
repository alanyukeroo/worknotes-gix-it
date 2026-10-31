# GIX GPU Cluster — Student Access Rollout and Reliability Testing

**Dates:** 2026-10-16, 2026-10-23, 2026-10-30 (Friday shifts)
**Location:** GIX GPU cluster: 1 controller and 5 GPU nodes, each with an RTX 5090
**Scope:** Slurm, rootless Podman, VS Code access, `gix` command-line tool, student and IT staff docs, reliability testing
**Status:** 🟡 In progress. Students can run jobs, and reliability testing is still under way
**Time:** 3 shifts, 8 hours each, 24 hours in total. Logs and commits show about 14.25 hours of that directly (see [Time Spent](#time-spent))
**Tools:** Worked with Claude Code. Root changes on the cluster were run with admin approval

---

## Summary

Students needed a simple way to train on the GPU cluster without learning Slurm, SSH to specific nodes, or container setup.

Over three shifts I:

- Set up VS Code devcontainers on all 5 GPU nodes, running through Slurm with rootless Podman.
- Built `gix`, a command-line tool students run from their laptop to upload, submit, follow and download jobs.
- Published the student guide and examples, including LeRobot, in `UW-GIX/gix-gpu-cluster`.
- Wrote an IT staff guide for monitoring, maintenance and troubleshooting.
- Started reliability testing, which found a real LeRobot failure before students hit it.

Students can now run a job with:

```bash
gix upload lerobot-act
gix submit lerobot-act/train.sbatch
gix logs <job> -f
gix download lerobot-act/outputs
```

---

## Time Spent

| Shift | Main work | Shift length | Shown in logs and commits |
| --- | --- | --- | --- |
| Fri 2026-10-16 | Devcontainer rollout, first SSSD incident, VS Code extension | 8 h | About 3 h (09:45–12:53) |
| Fri 2026-10-23 | SSSD fix, `gix` tool, student repo, examples, first admin guide | 8 h | About 3.5 h (11:17–14:54) |
| Fri 2026-10-30 | LeRobot fix, guides, IT staff doc, reliability testing, Microduck | 8 h | About 7.75 h (09:15–17:00, planned) |
| **Total** | | **24 h** | **About 14.25 h** |

The log figure is only what timestamps can prove. It doesn't cover planning, meetings, testing from my laptop, or other work during the shift.

---

## Timeline

### Shift 1: 2026-10-16 — Devcontainers on the Cluster

1. **Slurm access:** added a script that finds or starts a student's job, then connects them to the node running it. Jobs end after 30 idle minutes.
2. **Node login rules:** users without a job on a node are now refused. Sessions join the job, so its CPU, RAM and GPU limits apply.
3. **Containers:** set up rootless Podman with per-user image storage and GPU access through CDI. There's no Docker daemon or root socket.
4. **New users:** a Slurm prolog creates home folders, assigns container user ID ranges, and clears stale container state.
5. **Rollout:** piloted on one node, then rolled out one node at a time, testing admin login and refusal of users without a job at each step.
6. **Node 627 driver:** the NVIDIA driver wasn't loading because the kernel modules didn't match the running kernel. Installed the matching package and rebooted.
7. **Controller GPU lock:** limited the controller's GPU to an admin group, since it sits outside Slurm. It took effect at a later reboot.
8. **SSSD incident:** NetID logins hung on the controller because the AD backend kept being killed by its watchdog. Extension 0.2.0 made it worse by opening a new SSH login for each status check. Fixed by clearing the SSSD cache.
9. **VS Code extension:** published 0.2.0, then 0.2.1, which reuses one SSH connection.
10. **Shared image store:** tested and dropped it, because it broke `apt-get` inside containers. Each student pulls images into their own storage instead.

**Result:** GPU in a container passed on 5 of 5 nodes, and end-to-end VS Code passed with a real NetID.

### Shift 2: 2026-10-23 — The `gix` Tool and Student Repo

1. **SSSD came back:** applied the second fix (`ad_enabled_domains`, `ignore_group_members`). `id` went back to about 0.1 s, with no watchdog kills in the logs since.
2. **No-extension path:** tested Remote-SSH and devcontainer from macOS.
3. **`gix` tool:** built the laptop client and the server command: `setup`, `upload`, `submit`, `status`, `logs`, `cancel`, `download`.
4. **Podman in batch jobs:** found that rootless Podman failed in batch jobs because of the login host's `XDG_RUNTIME_DIR`. `gix submit` works around it.
5. **Student repo:** wrote the guide and the CIFAR-10, LeRobot and custom-code examples, then published to GitHub, including `UW-GIX/gix-gpu-cluster`.
6. **Admin guide:** wrote a first version for IT staff.
7. **`gix download` fix:** it now creates the target folder first (client 0.2.1).

**Result:** install, upload, submit, follow logs and download all worked from macOS.

### Shift 3: 2026-10-30 — LeRobot Fix, Docs and Reliability Testing

1. **Status review:** confirmed no new nodes, and that the controller still isn't a compute node. Found that node 621 had been left drained since its 2026-10-26 upgrade, and that all hosts were down from about 23:00 on 10-27 to 15:47 on 10-28 (cause unknown).
2. **Synced `UW-GIX/gix-gpu-cluster`:** client 0.2.1 and the LeRobot example.
3. **Real LeRobot failure:** a test from my laptop failed because `:latest` of the LeRobot image had removed the `--wandb.enable` flag. Fixed by pinning the image by digest and switching to `--trackio.enable`. The next run succeeded.
4. **Student docs:** rewrote the LeRobot section step by step, fixed the checkpoint path, and added a "Which way should I train?" section.
5. **IT staff doc:** wrote a guide focused on monitoring, maintenance and troubleshooting.
6. **Reliability testing:** see below.
7. **Microduck robot training:** ran `pollen-robotics/microduck_rl` in a rootless container, recorded a video, and viewed the simulation live in a browser on a Mac. A longer training run finished at 17:48. In its 10-second simulated video, the robot stays upright and moves. It hasn't been tested on a real robot.
8. **Student guide:** added what the tests found: missing tools, resuming from a checkpoint, tested LeRobot policies, `--mem`, and new troubleshooting entries.

---

## Reliability Test Results

As of 17:50 on 2026-10-30. Tests ran on free nodes only, with a test account and time limits.

| Test | Result | Notes |
| --- | --- | --- |
| LeRobot ACT, 2,000 steps, from a Mac | ✅ | Job 526 |
| LeRobot ACT with a quarter GPU (`shard:1`) | ✅ | Under 4 minutes including image pull |
| CIFAR-10 on 2 nodes | ✅ | 91.5% and 91.1% test accuracy |
| Own code from the template | ✅ | GPU, `requirements.txt` and data folder all worked |
| LeRobot ACT, full 50,000 steps | ✅ | About 25 minutes; checkpoint reached the NAS during the run |
| Job passes its time limit | ✅ ⚠️ | Stops cleanly, but work since the last checkpoint is lost |
| Resume from a checkpoint | ✅ | Works, and is now in the student guide |
| Four jobs on one GPU | ✅ | About 12 steps/s each, compared with 26–37 alone |
| `gix cancel` during training | ✅ | Other jobs carried on |
| More jobs than GPU shares | ✅ | Extra jobs queued and finished on their own |
| Bad commands and failing jobs | ✅ ⚠️ | Clear errors in most cases |
| NAS load from four jobs | ✅ ⚠️ | No errors; about 116 MB/s write in total |
| RAM limit | ✅ ⚠️ | Enforced, but swap isn't limited |
| Job without `--mem` | ⚠️ | Blocks other jobs on the node |
| Jobs sharing GPU memory | ⚠️ | One large job can make another run out of memory |
| Diffusion, 300 steps | ✅ | 5.8 steps/s; 1.05 GB checkpoint |
| SmolVLA fine-tune, 50 steps | ✅ ⚠️ | Runs, but the log doesn't confirm the pretrained weights loaded |
| VQ-BeT, 200 steps | ✅ | 10.6 steps/s; 160 MB checkpoint |
| Multi-Task DiT, 200 steps | ✅ | 7.8 steps/s; 880 MB checkpoint |
| TD-MPC | ❌ | Not a cluster problem: it only accepts square images, and this dataset is 640x480 |
| pi0.5 fine-tune | ❌ | Needs Hugging Face access to the gated PaliGemma tokenizer |
| 10 rounds of LeRobot and CIFAR-10 | ✅ | 20 of 20 jobs completed, with steady run times |
| Long shard job, planned for 3 hours | ✅ | Ran 2 h 31 min, about 338,000 steps at a steady 39 steps/s, max 70 C. Stopped as designed when a real user needed the node, and their job started right away |
| Short soak on one node | ✅ ⚠️ | 26 of 26 jobs completed before it was stopped early to free the GPU. One CIFAR-10 job took 10 minutes instead of about 18 seconds, stuck in `pip install` |
| README `git` recipe | ✅ | Fails without it, works with it in 23 seconds |
| Microduck RL, 4,096 robots, 100 iterations | ✅ | Rootless container in Slurm: about 120,000 steps/s, 5.6 GB GPU memory |
| Microduck video from a job | ✅ | 10-second MP4 downloaded to a Mac. Needed `libEGL` added first |
| Microduck live in a browser | ✅ ⚠️ | Viser through an SSH tunnel, smooth on a Mac with nothing installed. The port is open without a password, so it isn't ready for students |
| Microduck long run, 8,192 robots, 6,000 iterations | ✅ | Finished in 2 h 42 min with no errors, at about 131,000 steps/s. Mean reward rose from -0.06 to 127.5, and about 5% of episodes ended in a fall over the last 300 iterations. Reward was already 126.6 by iteration 750, so a much shorter run may be enough |

**Not tested yet:**

- New-student walkthrough, and Windows
- Nodes 620 and 622 (in use, and not to be disturbed) and node 621 (drained)
- Several real NetIDs pulling images at once
- Reboots and NAS outage tests (needs a maintenance window)
- A 2–3 day soak test
- pi0, which is expected to need the same Hugging Face access as pi0.5

---

## Findings

| Finding | Impact | Suggested fix |
| --- | --- | --- |
| No job limits (`MaxTime`, `DefMemPerCPU`) | One job can hold a GPU indefinitely or block a node | Set limits in `slurm.conf` |
| GPU shares don't isolate memory | One student's job can crash another's | Explain in the guide |
| `gix status` doesn't show if a finished job succeeded | Students have to open every log | Show the result in `gix status` |
| Checkpoints save only every 20,000 steps | Hours of work can be lost at the time limit | Add `--save_freq` to the example |
| Resume isn't documented | Students may restart from zero | Add to the guide |
| Swap isn't limited | One large job can slow a node | Set `ConstrainSwapSpace` |
| NAS write speed is shared | Uploads and checkpoints slow down under load | Monitor, and review the network link |
| pi0.5 downloads about 14 GB, then fails without access | Slow failure, and NAS space used per project | Document it, and check access early |
| `gix upload` always sends the whole folder | Slow with large datasets | Use `rsync` |
| No sample LeRobot dataset | Students without data can't try it | Add a small one, if it's OK to share |
| Podman `level=error` line in some logs, on jobs that still ran | Can confuse students | Added to troubleshooting |
| Slow route to PyPI: about 150 KB/s, against about 6 MB/s for Hugging Face (one measurement) | Package installs can stall and hold a GPU, and may explain the 10-minute job | Measure again, then review the network or add a package cache |
| PyTorch image has no `git` or `libEGL` | GitHub dependencies and headless rendering fail | Documented in the student guide |
| Browser viewer port has no password | Anyone who can reach the node can open it | Secure it before offering it to students |
| NAS credential handling | Needs tightening | Tracked separately by IT |

---

## Lessons Learned

- Pin container images. `:latest` changed between one test and the next, and it would have broken the example for every student.
- Test as a student would. Running the published steps from a laptop showed problems that testing on the server didn't.
- More isn't always faster. Doubling Microduck from 4,096 to 8,192 robots only raised speed from about 120,000 to 131,000 steps/s.
- Other users come first during testing. Use free nodes only, set time limits, and give way when someone is waiting.
- One test script tried to cancel a real user's job by mistake. `gix` refused it and nothing was affected, but test scripts should only ever use the test account's own jobs.

---

## Next Steps

1. Finish the two running tests, and add their results here.
2. Return node 621 to service after a test job.
3. Run the new-student and Windows tests.
4. Set `MaxTime` and `DefMemPerCPU`, then test again.
5. Schedule a maintenance window for reboot and NAS tests, and run the soak test.
6. Copy the latest student guide changes to `UW-GIX/gix-gpu-cluster` as needed.
7. Move setup scripts and change notes into a private repo that IT staff can reach.
