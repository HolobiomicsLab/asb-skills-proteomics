---
name: cross-model-compute-cost-tabulation
description: Use when multiple network variants (e.g., SC-GAN vs 2D/3D/attention conditional
  GANs) have been trained under the same regime and you must compare their wall-clock
  training burden per epoch and to plateau.
when_to_use_negative:
- Inputs are only evaluation metrics (normalized RMS error, peak SNR, structural similarity)
  with no training timing data — this skill measures cost, not prediction quality
- Only a single network is trained — cross-model cost comparison requires at least
  two models trained under comparable conditions
- The goal is memory (GPU RAM) profiling rather than wall-clock training time, although
  the paper flags both as limitations of SC-GAN
tools:
- name: SC-GAN
  role: Model under test whose per-epoch and time-to-plateau cost is measured
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline model included in the cost comparison table
- name: 3D conditional GAN
  role: Baseline model included in the cost comparison table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:377104ac4ac11392959449c0e42f6dc65457570df75cae10afc80775f852f094
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - SC-GAN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-model-compute-cost-tabulation

## Summary

Tabulates wall-clock training cost (minutes per epoch and minutes to reach the training plateau) across competing neural network variants, as done when quantifying SC-GAN's computational burden against the 2D cGAN, 3D cGAN, and Att cGAN baselines.

## When to use

Use when multiple network variants (e.g., SC-GAN vs 2D/3D/attention conditional GANs) have been trained under the same regime and you must compare their wall-clock training burden per epoch and to plateau.

## When NOT to use

- Inputs are only evaluation metrics (normalized RMS error, peak SNR, structural similarity) with no training timing data — this skill measures cost, not prediction quality
- Only a single network is trained — cross-model cost comparison requires at least two models trained under comparable conditions
- The goal is memory (GPU RAM) profiling rather than wall-clock training time, although the paper flags both as limitations of SC-GAN

## Inputs

- Trained model checkpoints/loss curves from SC-GAN, 2D conditional GAN, 3D conditional GAN, and Att cGAN runs
- Wall-clock timing logs per epoch and cumulative training time from each run (TensorFlow on NVIDIA GPU cluster)
- Training hyperparameters (e.g., epochs=120 default, lr=0.001, g_reg=0.001, d_reg=0.001, l1_weight=200, B_weight=200) from training.py configuration

## Outputs

- time_cost_comparison.csv with one row per network (SC-GAN, 2D cGAN, 3D cGAN, Att cGAN) and columns for minutes per epoch and minutes to plateau

## How to apply

Train each network (SC-GAN and the 2D cGAN, 3D cGAN, and Att cGAN baselines) under identical conditions — for SC-GAN this means TensorFlow on an NVIDIA GPU cluster with the reported hyperparameters (lr=0.001 with cosine decay, g_reg/d_reg=0.001, l1_weight=200, B_weight=200, batch size=1) using the repository's training.py. Instrument each run to record wall-clock minutes per epoch and cumulative minutes until the training loss plateaus (~65 min/epoch and ~1300 min to plateau for SC-GAN). Aggregate the timings into a single per-model table with columns for minutes per epoch and minutes to plateau, saved as time_cost_comparison.csv, so computational cost can be weighed against the accuracy gains reported via normalized RMS error, peak SNR, and structural similarity.

## Related tools

- **SC-GAN** (Model under test whose per-epoch and time-to-plateau cost is measured) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline model included in the cost comparison table)
- **3D conditional GAN** (Baseline model included in the cost comparison table)

## Examples

```
python training.py --trainig_size= --gpu= --epoches= --img_size= --data_dir= --modalities= --logdir=
```

## Evaluation signals

- time_cost_comparison.csv exists and contains exactly one row per network: SC-GAN, 2D cGAN, 3D cGAN, Att cGAN
- Recorded SC-GAN timings match the reported ~65 minutes per epoch and ~1300 minutes to reach the training plateau
- SC-GAN's training cost is on par with the Att cGAN baseline (i.e., the self-attention/stabilization additions do not add training time beyond the 3D baseline class)
- All four models were trained under the same regime (same data, GPU cluster, hyperparameter defaults such as lr=0.001 and epochs=120) so timings are comparable
- Timings are large enough to reflect the paper's stated computational burden: lengthy training and heavy GPU resource requirements

## Limitations

- Computational cost is the main limitation of SC-GAN: lengthy training and heavy computational resources such as GPU memory make the timing experiment expensive to reproduce
- Timings are hardware- and software-dependent (TensorFlow version and NVIDIA GPU cluster used in the study), so absolute minutes may not transfer to other setups
- The extracted article index does not state the exact 65 min/epoch and 1300 min plateau figures; those numbers come from the workflow card and should be re-verified against the full text or a rerun

## Evidence
