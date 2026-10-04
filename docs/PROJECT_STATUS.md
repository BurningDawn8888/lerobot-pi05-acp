# Project Status

Last updated: 2026-10-04

## Current milestone

The reference implementation is at Stage 8: second-round value-model training preparation.

Completed milestones:

1. LeRobot 0.6.2 and Pi0.5 ACP compatibility layer.
2. Complete human-in-the-loop rollout collection and immutable Round 1 dataset freeze.
3. Round 1 trajectory-value training and per-frame advantage inference.
4. Pi0.5 ACP Round 1 fine-tuning with binary advantage conditioning.
5. Fixed real-robot comparison between the frozen baseline and ACP Round 1.
6. Targeted Round 2 collection from Round 1 failure conditions.
7. Round 2 quality control and immutable aggregate training-pool creation.

The combined local training pool contains 204 complete episodes and 239,093 frames. It includes 127 successful and 77 failed episodes. Both camera streams and representative H.264 and AV1 source episodes were decoded through the training reader.

## Next operations

1. Run a short Round 2 value-model smoke test.
2. Train and validate the formal Round 2 value model.
3. Infer `value_round2`, `advantage_round2`, and `acp_indicator_round2` on a dataset copy.
4. Train Pi0.5 ACP Round 2 without overwriting Round 1 artifacts.
5. Re-evaluate the baseline, ACP Round 1, and ACP Round 2 on the same fixed condition matrix.

## Promotion rule

No new checkpoint replaces the frozen production baseline unless it improves autonomous success while preserving or improving wrong-object, placement, and safety metrics. Training loss alone is never a promotion criterion.

## Repository boundary

This public repository contains source code and English documentation only. It does not contain datasets, model checkpoints, videos, robot calibration, reset poses, experiment logs, credentials, private repository identifiers, or machine-specific configuration.
