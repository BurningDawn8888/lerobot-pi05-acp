# LeRobot Pi0.5 ACP

An experimental, English-only LeRobot 0.6.2 fork for improving Pi0.5 robot policies with a human-in-the-loop reinforcement-learning data loop.

The implementation adds:

- value-model training with `lerobot-value-train`;
- offline value and n-step advantage inference with `lerobot-value-infer`;
- binary Advantage-Conditioned Policy (ACP) labels stored in LeRobot datasets;
- ACP prompt conditioning and configurable indicator dropout during Pi0.5 training;
- intervention-aware annotations and dataset statistics;
- checkpoint-safe resume and Windows `last` junction replacement.

This repository contains source code only. It intentionally excludes robot calibration files, datasets, model weights, experiment outputs, logs, credentials, and machine-specific configuration.

## Project status

This is research software, not a production safety controller. The workflow has been exercised with an SO-101 follower/leader setup, two OpenCV cameras, Pi0.5, Windows, CUDA, and a single 32 GB GPU. Hardware behavior must be validated locally with an accessible physical emergency stop.

## Workflow

1. Start from a working Pi0.5 behavior-cloning checkpoint.
2. Collect complete autonomous, intervention-recovery, and failed trajectories.
3. Freeze and quality-check the rollout dataset.
4. Train a trajectory value model.
5. Infer per-frame value, n-step advantage, and a binary ACP indicator on a dataset copy.
6. Fine-tune Pi0.5 with ACP enabled and indicator dropout.
7. Compare the baseline and ACP checkpoints with the same fixed real-robot evaluation matrix.
8. Freeze the winning checkpoint and repeat the loop on newly discovered failures.

See [ACP workflow](docs/ACP_WORKFLOW.md) for command templates and acceptance checks.

## Installation

Python 3.12 or newer is required. Create an isolated environment and install the repository in editable mode:

```bash
conda create -n lerobotrl python=3.12 -y
conda activate lerobotrl
pip install -e ".[core_scripts]"
```

Install the PyTorch build appropriate for your CUDA driver before starting GPU training. Follow the upstream LeRobot hardware installation instructions for the robot and cameras.

Verify the added commands:

```bash
lerobot-value-train --help
lerobot-value-infer --help
lerobot-train --help
```

## Minimal ACP policy training template

```bash
lerobot-train \
  --policy.path=/path/to/pi05_checkpoint/pretrained_model \
  --dataset.repo_id=YOUR_USER/YOUR_ACP_DATASET \
  --dataset.root=/path/to/acp_dataset \
  --acp.enable=true \
  --acp.indicator_field=complementary_info.acp_indicator \
  --acp.indicator_dropout_prob=0.30 \
  --policy.device=cuda \
  --policy.dtype=bfloat16 \
  --policy.gradient_checkpointing=true \
  --policy.compile_model=false \
  --policy.freeze_vision_encoder=false \
  --policy.train_expert_only=true \
  --batch_size=8 \
  --steps=15000 \
  --save_freq=3000 \
  --output_dir=outputs/pi05_acp
```

Field names are configurable. Do not run ACP policy training until the indicator field has been validated as binary and aligned with the original frames and actions.

## Repository boundaries

Keep these items outside Git: datasets, outputs, checkpoints, W&B runs, logs, local configuration, credentials, robot calibration, reset poses, raw recordings, and downloaded model weights.

Publish datasets and trained checkpoints separately through repositories with explicit licenses and dataset/model cards.

## Attribution

This project is derived from Hugging Face LeRobot and retains its Apache License 2.0. The reinforcement-learning workflow was informed by the open-source Evo-RL project from MINT-SJTU. See [NOTICE](NOTICE) and [UPSTREAM_README.md](UPSTREAM_README.md).

## License

Apache License 2.0. See [LICENSE](LICENSE).
