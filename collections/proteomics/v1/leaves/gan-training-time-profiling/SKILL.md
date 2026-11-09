---
name: gan-training-time-profiling
description: Use when benchmarking training wall-clock cost of GAN models for multimodal
  neuroimaging synthesis (e.g., T1w+FLAIR to PET/FA/MD tasks) and you need minutes-per-epoch
  and time-to-plateau per architecture.
when_to_use_negative:
- The analysis goal is synthesis accuracy (nRMSE, PSNR, SSIM) rather than computational
  cost — use evaluation of synthesized images instead of timing
- Input task is 2D image synthesis or a non-neuroimaging domain where the 256x256x256
  3D volumes and per-epoch timing profile of SC-GAN do not transfer
- Hardware differs from the study's NVIDIA GPU setup and cross-model timing comparability
  cannot be guaranteed
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Model whose training time is profiled (65 min/epoch, ~1300 min to plateau)
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline architecture timed for comparison of per-epoch and plateau cost
- name: 3D conditional GAN
  role: Baseline architecture timed for comparison of per-epoch and plateau cost
- name: TensorFlow
  role: Training framework (TensorFlow 1.12.2) in which runs are instrumented for
    wall-clock timing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:a259dd478f612323c74e862f99d857047942ea23d71cfe30ae308967d150354a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - SC-GAN
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gan-training-time-profiling

## Summary

Profiles the wall-clock training cost of a 3D conditional GAN (SC-GAN) against baseline GAN architectures (2D cGAN, 3D cGAN, Att cGAN) by instrumenting each run for minutes per epoch and cumulative minutes to reach the training loss plateau, exposing the computational burden reported as SC-GAN's main limitation.

## When to use

Use when benchmarking training wall-clock cost of GAN models for multimodal neuroimaging synthesis (e.g., T1w+FLAIR to PET/FA/MD tasks) and you need minutes-per-epoch and time-to-plateau per architecture.

## When NOT to use

- The analysis goal is synthesis accuracy (nRMSE, PSNR, SSIM) rather than computational cost — use evaluation of synthesized images instead of timing
- Input task is 2D image synthesis or a non-neuroimaging domain where the 256x256x256 3D volumes and per-epoch timing profile of SC-GAN do not transfer
- Hardware differs from the study's NVIDIA GPU setup and cross-model timing comparability cannot be guaranteed

## Inputs

- Preprocessed ADNI-3 training volumes (train/val/test directory structure with .nii.gz modality files, e.g., flair_t1w_av45)
- Training script ./training with configurable --epoches, --img_size, --lr, --g_reg, --d_reg, --l1_weight, --B_weight flags
- NVIDIA GPU on the study's compute cluster with TensorFlow 1.12.2 environment (requirements.txt)

## Outputs

- time_cost_comparison.csv: one row per network (SC-GAN, 2D cGAN, 3D cGAN, Att cGAN) with minutes per epoch and minutes to plateau
- TensorBoard loss/time logs (--logdir) documenting convergence plateau per model

## How to apply

Train SC-GAN and each baseline (2D conditional GAN, 3D conditional GAN, Att cGAN) with the paper's hyperparameters (SC-GAN: l1_weight/B-rmse weight 200, spectral-normalization lambda 20, generator/discriminator regularizers 0.001, lr 0.001 with cosine decay, batch size 1, 256x256x256 volumes, default 120 epochs). Instrument every run with wall-clock timers per epoch and accumulate time until the training loss plateaus (approximately 1300 minutes for SC-GAN, about 65 minutes per epoch); TensorBoard logs (--logdir) provide the loss curves needed to detect the plateau. Aggregate the four networks' timings into a single table (time_cost_comparison.csv) with one row per model and columns for minutes per epoch and minutes to plateau. Run on the same NVIDIA GPU configuration used in the study (TensorFlow 1.12.2, python 3.6.9, requirements from requirements.txt) so timings are comparable across models.

## Related tools

- **SC-GAN** (Model whose training time is profiled (65 min/epoch, ~1300 min to plateau)) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline architecture timed for comparison of per-epoch and plateau cost)
- **3D conditional GAN** (Baseline architecture timed for comparison of per-epoch and plateau cost)
- **TensorFlow** (Training framework (TensorFlow 1.12.2) in which runs are instrumented for wall-clock timing)

## Examples

```
python training.py --trainig_size= --gpu= --epoches= --img_size= --data_dir= --modalities= --logdir=
```

## Evaluation signals

- time_cost_comparison.csv exists with exactly one row per network (SC-GAN, 2D cGAN, 3D cGAN, Att cGAN) and columns for minutes per epoch and minutes to plateau
- SC-GAN timing reproduces the reported values: ~65 minutes per epoch and ~1300 minutes to reach the training plateau
- SC-GAN's time cost matches the Att cGAN baseline while remaining markedly heavier than the 2D cGAN
- Plateau detection is consistent: the recorded time-to-plateau corresponds to where the TensorBoard training loss curve flattens, not an arbitrary epoch count
- All four runs used identical hardware (NVIDIA GPU) and hyperparameter settings so timings are directly comparable

## Limitations

- Computational cost is the main limitation of SC-GAN: it requires lengthy training and heavy computational resources, such as GPU memory
- Timings are hardware- and software-version-specific (TensorFlow 1.12.2, python 3.6.9 per requirements.txt) and do not generalize to other GPUs or frameworks
- Training cost was profiled for the ADNI-3 synthesis tasks (PET, FA, MD from T1w+FLAIR); other datasets or image sizes will yield different epoch times

## Evidence
