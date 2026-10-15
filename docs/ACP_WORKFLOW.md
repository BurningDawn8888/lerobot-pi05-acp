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

## 7. Build the next rollout round

Do not train directly from evaluation CSV files or summary logs. Use failed evaluation conditions only to define targeted tasks, then record complete LeRobot episodes from the initial robot state through the terminal outcome.

For every new episode:

- record all configured camera streams, robot state, policy action, and executed action;
- assign one terminal result such as success, grasp failure, placement failure, wrong object, or abnormal termination;
- finish video encoding and dataset finalization before closing the recorder;
- preserve the raw round as an immutable source dataset;
- keep failed trajectories for value learning even when they are excluded from behavior-cloning subsets.

## 8. Validate and combine immutable rounds

Create a new training-pool directory or Hub revision instead of appending to an already trained revision. Preserve original episode outcomes and add a source-round identifier so later analyses can separate distribution changes from model changes.

Acceptance checks:

- episode indices are continuous and unique after aggregation;
- every episode has exactly one terminal outcome and one binary success label;
- both camera streams decode at the beginning and end of representative episodes from every source round;
- frame counts, action dimensions, state dimensions, task text, and FPS match the declared metadata;
- the aggregate success/failure counts equal the sum of the source manifests;
- no original dataset, video, action, or Round 1 ACP field is overwritten.

Mixed H.264 and AV1 sources can remain in separate MP4 files when the training reader probes each file directly. Treat codec and pixel-format metadata as encoder metadata during aggregation, record heterogeneous values as unspecified, and test real decoding from each codec range before training.

## 9. Train subsequent ACP rounds

Use a new value checkpoint, field suffix, output directory, and model repository for each round. For example, Round 2 should write `value_round2`, `advantage_round2`, and `acp_indicator_round2`; it must not overwrite Round 1 fields.

Run a short smoke test before every full job. Confirm video decoding, finite loss, expected GPU memory, checkpoint creation, and unchanged source data. Then repeat Sections 2 through 6 using the combined frozen training pool and the new field suffix.

Document which checkpoint initializes the next ACP policy. A baseline initialization provides a cleaner comparison, while an accepted prior ACP checkpoint continues the learned policy; the choice must not be implicit.

## 10. Current reference milestone

The reference SO-101 project has completed Round 2 value training, advantage inference, 15,000-step ACP training, and fixed-matrix evaluation. On the same 81-task matrix, the Pi0.5 baseline, ACP R1, and ACP R2 achieved 77.8%, 60.5%, and 67.9% autonomous success. R2 reduced initial grasp failures relative to R1 but remained below the baseline and exposed grasp-slip failures during early transport.

Frame-by-frame review produced an R3 grasp-stability correction dataset with 25 episodes and 31,527 frames. R3 value training completed 8,000 steps, and full-dataset inference wrote `value_round3`, `advantage_round3`, and `acp_indicator_round3` to a dataset copy. The indicator is finite and binary for every frame; the observed positive ratio is 41.82%, and all 5,521 intervention frames are positive. A 20-step R3 ACP smoke test passed. The current operation is formal 15,000-step R3 ACP training followed by fixed-condition real-robot evaluation.

This sequence is a targeted correction branch, not evidence that the complete π*0.6 loop has been reproduced. A complete next data loop requires an R3 evaluation followed by another collection and training round. Exceeding the baseline is a promotion gate, not a prerequisite for initiating another diagnosable correction round.

These counts document one experiment and are not bundled with this source repository. Public datasets and released models are listed in [OPEN_RESOURCES.md](OPEN_RESOURCES.md). Recordings, model weights, hardware calibration, credentials, and machine-specific state remain outside Git.
