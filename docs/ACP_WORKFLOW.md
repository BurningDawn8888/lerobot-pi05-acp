# Pi0.5 Advantage-Conditioned Policy Workflow

Replace every uppercase placeholder with your own path or Hub repository identifier.

## 1. Prepare and freeze rollout data

Keep complete episodes from the initial state through the terminal outcome. Include autonomous successes, successful intervention recoveries, and autonomous failures. Preserve observations, policy actions, executed actions, intervention state, and an episode-level success label. Work on a copy before adding value or advantage columns.

Acceptance checks:

- both camera streams and robot state are present for every frame;
- timestamps and actions remain aligned;
- episode outcomes are explicit and auditable;
- no episode starts at the intervention point;
- the frozen source dataset has an immutable revision or checksum.

## 2. Train the value model

```bash
lerobot-value-train \
  --dataset.repo_id=YOUR_USER/YOUR_ROLLOUT_DATASET \
  --dataset.root=/path/to/frozen_rollouts \
  --value.type=pistar06 \
  --value.dtype=bfloat16 \
  --targets.success_field=episode_success \
  --targets.default_success=failure \
  --targets.c_fail_coef=1.0 \
  --batch_size=8 \
  --num_workers=0 \
  --steps=8000 \
  --save_freq=2000 \
  --output_dir=outputs/value_round1
```

Inspect held-out successful, recovered, and failed episodes. Reject a value model that tracks lighting, background, camera motion, or operator appearance instead of task progress.

## 3. Infer value, advantage, and ACP labels

```bash
lerobot-value-infer \
  --dataset.repo_id=YOUR_USER/YOUR_ROLLOUT_DATASET \
  --dataset.root=/path/to/rollouts_acp_copy \
  --inference.checkpoint_path=/path/to/value_round1 \
  --inference.checkpoint_ref=last \
  --runtime.device=cuda \
  --runtime.batch_size=8 \
  --runtime.num_workers=0 \
  --acp.enable=true \
  --acp.n_step=50 \
  --acp.positive_ratio=0.30 \
  --acp.force_intervention_positive=true \
  --acp.intervention_field=complementary_info.is_intervention \
  --acp.value_field=complementary_info.value_round1 \
  --acp.advantage_field=complementary_info.advantage_round1 \
  --acp.indicator_field=complementary_info.acp_indicator_round1 \
  --acp.c_fail_coef=1.0 \
  --output_dir=outputs/value_infer_round1
```

Verify that all three generated fields cover every frame, the indicator is binary, positive ratios are plausible globally and per task, and original observations, actions, videos, and outcomes are unchanged.

## 4. Fine-tune Pi0.5 with ACP

```bash
lerobot-train \
  --policy.path=/path/to/best_pi05_baseline/pretrained_model \
  --dataset.repo_id=YOUR_USER/YOUR_ROLLOUT_DATASET \
  --dataset.root=/path/to/rollouts_acp_copy \
  --acp.enable=true \
  --acp.indicator_field=complementary_info.acp_indicator_round1 \
  --acp.indicator_dropout_prob=0.30 \
  --policy.device=cuda \
  --policy.dtype=bfloat16 \
  --policy.gradient_checkpointing=true \
  --policy.compile_model=false \
  --policy.freeze_vision_encoder=false \
  --policy.train_expert_only=true \
  --batch_size=8 \
  --num_workers=0 \
  --steps=15000 \
  --save_freq=3000 \
  --resume=false \
  --output_dir=outputs/pi05_acp_round1
```

Do not assume the final checkpoint is the best checkpoint. Preserve intermediate checkpoints and compare them on hardware.

## 5. Resume safely

```bash
lerobot-train \
  --config_path=/path/to/checkpoints/006000/pretrained_model/train_config.json \
  --resume=true
```

Keep batch size and distributed topology unchanged when sample-exact resume behavior matters.

## 6. Fixed real-robot evaluation

Evaluate the original Pi0.5 baseline and ACP checkpoints with identical instructions, placements, calibration, lighting, duration, and safety procedure. Record full success, wrong-object selection, grasp failure, placement failure, completion time, intervention requirement, and every safety event.

Promote an ACP checkpoint only when autonomous success improves without increasing wrong-object or safety-event rates.
