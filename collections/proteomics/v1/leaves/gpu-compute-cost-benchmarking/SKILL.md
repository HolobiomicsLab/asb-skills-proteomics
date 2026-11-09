---
name: gpu-compute-cost-benchmarking
description: Use when comparing wall-clock training cost (minutes per epoch, minutes
  to plateau, GPU memory burden) of a new GAN or network against baseline models on
  a fixed GPU cluster.
when_to_use_negative:
- The goal is synthesis accuracy (normalized RMS error, peak SNR, structural similarity)
  rather than computational cost — use the evaluation-metric workflow instead
- Runs are executed on heterogeneous hardware or mixed GPU models, making wall-clock
  comparisons between networks invalid
- The model has not yet reached a stable training regime (SC-GAN relies on spectral
  normalization and feature matching for convergence), so 'time to plateau' is undefined
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Model being benchmarked; trained via training.py to measure minutes per epoch
    and time to plateau
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline network whose per-epoch and plateau training times are recorded for
    comparison
- name: 3D conditional GAN
  role: Baseline network whose per-epoch and plateau training times are recorded for
    comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:c3d3f51cb50a7ba9a3fcb38f2ca5441b40749c03cde3be442094f81293cb9174
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - SC-GAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gpu-compute-cost-benchmarking

## Summary

Benchmarks the wall-clock training cost of a deep-learning model (here SC-GAN, a 3D self-attention conditional GAN with spectral normalization) against baselines by recording minutes per epoch and minutes to reach the training plateau, exposing the computational burden that motivated the paper's own limitation statement.

## When to use

Use when comparing wall-clock training cost (minutes per epoch, minutes to plateau, GPU memory burden) of a new GAN or network against baseline models on a fixed GPU cluster.

## When NOT to use

- The goal is synthesis accuracy (normalized RMS error, peak SNR, structural similarity) rather than computational cost — use the evaluation-metric workflow instead
- Runs are executed on heterogeneous hardware or mixed GPU models, making wall-clock comparisons between networks invalid
- The model has not yet reached a stable training regime (SC-GAN relies on spectral normalization and feature matching for convergence), so 'time to plateau' is undefined

## Inputs

- Training runs of SC-GAN, 2D conditional GAN, 3D conditional GAN, and Att cGAN under matched hyperparameters (lr=0.001, l1_weight=200, B_weight=200, g_reg=d_reg=0.001, batch size=1)
- NVIDIA GPU cluster with TensorFlow 1.12.2 environment (python 3.6.9, requirements.txt from the SC-GAN repo)
- Multimodal MRI training data (T1w and FLAIR inputs, e.g., flair_t1w_av45) organized per-subject in train/val/test directories

## Outputs

- time_cost_comparison.csv with one row per network (SC-GAN, 2D cGAN, 3D cGAN, Att cGAN) giving minutes per epoch and minutes to plateau
- Per-run timing logs of wall-clock time per epoch and cumulative training time

## How to apply

Train SC-GAN and the baselines (2D conditional GAN, 3D conditional GAN, Att cGAN) under identical conditions: TensorFlow 1.12.2 on an NVIDIA GPU cluster with the reported hyperparameters (l1_weight μ=200, B-rmse weight γ=200, λ=20, generator/discriminator regularizers νG=νD=0.001, batch size=1, initial learning rate 0.001 with cosine decay). Instrument each run to record wall-clock time per epoch and cumulative time until the training loss plateaus. For this paper SC-GAN cost ~65 minutes per epoch and ~1300 minutes to plateau, the same training time cost as the Att cGAN baseline. Aggregate the per-model timing measurements (minutes per epoch and minutes to plateau) into a single table saved as time_cost_comparison.csv. The rationale is to quantify the paper's stated limitation that SC-GAN 'requires lengthy training and heavy computational resources, such as GPU memory.'

## Related tools

- **SC-GAN** (Model being benchmarked; trained via training.py to measure minutes per epoch and time to plateau) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline network whose per-epoch and plateau training times are recorded for comparison)
- **3D conditional GAN** (Baseline network whose per-epoch and plateau training times are recorded for comparison)

## Examples

```
python training.py --trainig_size= --gpu= --epoches= --img_size= --data_dir= --modalities= --logdir=
```

## Evaluation signals

- time_cost_comparison.csv exists and contains exactly one row per network: SC-GAN, 2D cGAN, 3D cGAN, Att cGAN
- SC-GAN timing entries fall at approximately 65 minutes per epoch and 1300 minutes to plateau
- SC-GAN and Att cGAN rows show comparable training time cost, matching the paper's reported equivalence
- All four networks were trained with identical hyperparameters (lr=0.001, l1_weight=200, B_weight=200, g_reg=d_reg=0.001) so timings are directly comparable
- Recorded times reflect the paper's reported burden: lengthy training and heavy GPU memory requirements

## Limitations

- Computational cost is the main limitation of SC-GAN: it requires lengthy training (~1300 minutes to plateau) and heavy computational resources such as GPU memory, which limits replication on smaller hardware
- Timing measurements are hardware- and software-dependent (TensorFlow 1.12.2, specific NVIDIA GPU cluster) and do not transfer directly to other environments
- Time to plateau depends on the loss stabilizing; without the spectral normalization and feature matching used in SC-GAN, baseline runs may not converge comparably

## Evidence
