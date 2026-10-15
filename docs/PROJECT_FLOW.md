# Project Flow and Method Provenance

## Implemented project path

Solid arrows represent completed or explicitly scheduled project stages. The dashed arrow is a future conditional round, not an executed loop.

```mermaid
flowchart TD
    S0[0 Strategy and environment baseline] --> S1[1 Complete HIL rollout collection]
    S1 --> S2[2 Dataset stratification and manifests]
    S2 --> S3[3 Round 1 value training]
    S3 --> S4[4 Round 1 advantage inference and binary labels]
    S4 --> S5[5 Pi0.5 ACP R1 training]
    S5 --> S6[6 Baseline vs R1 fixed evaluation]
    S6 --> S7[7 Targeted R2 collection]
    S7 --> S8[8 R2 value advantage training and reevaluation]
    S8 --> S9[9 Frame-by-frame R2 failure review]
    S9 --> S10[10 R3 grasp-stability collection]
    S10 --> S11[11 R3 value training and advantage inference]
    S11 --> S12[12 R3 ACP formal training]
    S12 --> S13[13 R3 fixed-condition real-robot evaluation]
    S13 --> G{Pass promotion gates?}
    G -->|Yes| P[Freeze candidate and make a release decision]
    G -.->|No and failures are actionable| S9
```

The implemented evidence currently ends at R3 formal training preparation. A complete next-round data loop requires R3 evaluation, another targeted collection, and another value/ACP training round.

## Relationship to π*0.6 and Evo-RL

```mermaid
flowchart LR
    A[π*0.6 / Evo-RL method concepts] --> B[Value learning]
    A --> C[Advantage filtering]
    A --> D[Conditioned policy update]
    B --> E[Pi0.5 + SO-101 implementation]
    C --> E
    D --> E
    E --> F[Human takeover and result confirmation]
    E --> G[Complete dual-camera LeRobot episodes]
    E --> H[Safe homing and hardware error handling]
    E --> I[Fixed-matrix real-robot evaluation]
    E --> J[Frame-level failure review and targeted recollection]
```

This repository borrows the value-learning, advantage-filtering, and conditional-update mechanisms. It does not claim to reproduce the complete π*0.6 algorithm, automatic continuous rollout, or the paper's reported results.

## Decision rule

- Better than the frozen baseline and safe: eligible for promotion.
- Weaker than the baseline but failures are reliable and actionable: eligible for another targeted improvement round, not promotion.
- Calibration, homing, communication, labeling, or data integrity is unreliable: stop and repair the system before collecting more training data.
