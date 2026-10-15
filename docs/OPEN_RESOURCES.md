# Open Resources

This page maps the public source repository to released datasets and model artifacts. Publication does not imply that every checkpoint passed the production promotion gate.

## Source code

| Resource | URL | Scope |
|---|---|---|
| LeRobot Pi0.5 ACP | https://github.com/BurningDawn8888/lerobot-pi05-acp | LeRobot 0.6.2 compatibility layer, value training and inference, ACP policy training, HIL collection, evaluation utilities, and English documentation |

## Datasets

| Dataset | Immutable revision | Purpose |
|---|---|---|
| https://huggingface.co/datasets/ted88168/rollout_colorlogo_rl_round1 | `b361a8f9d952cc082dc3f6d0a195ca05a24435e0` | Round 1 rollout training data |
| https://huggingface.co/datasets/ted88168/rollout_colorlogo_rl_round2_combined | `6bd6aee1df33cdfb1dbc01d4ca708756331e6073` | Frozen 204-episode Round 1 + Round 2 training pool |
| https://huggingface.co/datasets/ted88168/rollout_colorlogo_acp_r2_full_eval_resident | `de93de72225f56f31f7d52d6294d6036a6f173da` | Frozen 243-episode baseline/R1/R2 fixed-matrix evaluation evidence |
| https://huggingface.co/datasets/ted88168/rollout_colorlogo_r3_grasp_stability_v1 | `d576d5885dd38b521c34050f4ecdb3ee230cca74` | R3 grasp-stability correction data: 25 episodes and 31,527 frames |

Evaluation data remains evaluation-only. It must not be merged back into a training pool. Value and advantage inference must run on a local copy rather than mutating an immutable source revision.

## Value models

| Model | Status | Purpose |
|---|---|---|
| https://huggingface.co/ted88168/colorlogo_value_round1 | Released | Round 1 trajectory-value model |
| https://huggingface.co/ted88168/colorlogo_value_round2 | Released | Round 2 trajectory-value model |
| https://huggingface.co/ted88168/colorlogo_value_r3_grasp_stability_v1 | Released | R3 grasp-stability value model, 8,000-step checkpoint |

## ACP policy models

| Model | Status | Evidence boundary |
|---|---|---|
| https://huggingface.co/ted88168/pi05_colorlogo_acp_round1 | Released | Evaluated on the fixed 81-task matrix; 60.5% autonomous success |
| https://huggingface.co/ted88168/pi05_colorlogo_acp_round2 | Released | Evaluated on the fixed 81-task matrix; 67.9% autonomous success |
| R3 ACP candidate | Not released | Formal training and fixed-condition evaluation must finish before a publication decision |

## Reproducibility boundary

The public artifacts support inspection of schemas, labels, command entry points, training configuration, evaluation evidence, and released checkpoints. They do not replace robot calibration, camera placement, homing accuracy, hardware safety procedures, or the physical evaluation environment.

Do not publish access tokens, local calibration files, serial ports, reset poses, raw operator recordings, W&B credentials, or machine-specific configuration.
