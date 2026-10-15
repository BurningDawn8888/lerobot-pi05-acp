# Project Status

Last updated: 2026-10-14

## Current milestone

The reference SO-101 project is at the R3 policy-training and fixed-condition evaluation boundary.

Completed work:

1. LeRobot 0.6.2 and Pi0.5 ACP compatibility layer.
2. Round 1 human-in-the-loop collection, quality control, value training, advantage inference, and 15,000-step ACP training.
3. Fixed-matrix comparison of the Pi0.5 baseline and ACP R1.
4. Targeted Round 2 collection and creation of a 204-episode, 239,093-frame combined training pool with 127 successful and 77 failed episodes.
5. Round 2 value training, full-dataset advantage inference, 15,000-step ACP training, and fixed-matrix evaluation.
6. A frozen 243-episode evaluation set covering 81 tasks each for the baseline, R1, and R2.
7. Frame-by-frame review of R2 placement failures, identifying grasp slip and early transport drop as the dominant failure modes.
8. R3 grasp-stability correction collection: 25 episodes and 31,527 frames, including 15 autonomous successes, 6 successful interventions, and 4 failures.
9. R3 value training for 8,000 steps and full-dataset value, advantage, and ACP-indicator inference. The observed positive indicator ratio is 41.82%; all 5,521 intervention frames are positive.
10. A 20-step R3 ACP smoke test.

Current operation:

- train the R3 ACP candidate for 15,000 steps from the accepted R2 15,000-step checkpoint;
- preserve checkpoints every 3,000 steps;
- evaluate R3 under the same fixed task, start-pose, homing, camera, and resident-model conditions before any promotion or release decision.

## Fixed-matrix evidence

| Policy | Checkpoint | Tasks | Autonomous success |
|---|---:|---:|---:|
| Pi0.5 baseline | 15,000 | 81 | 77.8% |
| Pi0.5 ACP R1 | 15,000 | 81 | 60.5% |
| Pi0.5 ACP R2 | 15,000 | 81 | 67.9% |

R2 improved over R1 but remained 9.9 percentage points below the baseline. It reduced initial grasp failures while increasing transport-time drops. These results support targeted failure-mode changes, not a claim that ACP already outperforms the baseline.

## Loop boundary

The implemented path is:

`R2 fixed evaluation -> failure review -> R3 targeted collection -> R3 value and advantage processing -> R3 ACP training`

The project has not yet completed:

`R3 fixed evaluation -> another collection round -> another value/ACP training round`

Automatic continuous rollout and a complete autonomous multi-round loop are not implemented. Exceeding the baseline is required for promotion, but not for entering another targeted correction round. If a candidate is weaker yet its failures are reliable and actionable, another round remains valid. Systemic calibration, homing, communication, or data errors must be repaired before additional training.

## Promotion rule

No checkpoint replaces the frozen production baseline unless it improves autonomous success while preserving or improving wrong-object, placement, homing, and safety metrics under the same evaluation conditions. Training loss alone is never a promotion criterion.

## Public evidence

Public source, dataset, model, and immutable revision references are maintained in [OPEN_RESOURCES.md](OPEN_RESOURCES.md).

## Repository boundary

This repository contains source code and English documentation only. It excludes datasets, model weights, videos, robot calibration, reset poses, experiment logs, credentials, tokens, private identifiers, and machine-specific configuration.
