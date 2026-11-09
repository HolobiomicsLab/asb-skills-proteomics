---
name: training-convergence-plateau-detection
description: Use when comparing the training wall-clock cost of generative models
  (minutes per epoch, time to plateau) or when you must document the computational
  burden of a 3D GAN before committing GPU resources.
when_to_use_negative:
- The goal is synthesis accuracy (normalized RMS error, peak SNR, structural similarity)
  rather than training-cost benchmarking.
- Only a single network is available, so no cross-model timing comparison (SC-GAN
  vs. Att cGAN, etc.) can be formed.
- Input is a pre-trained checkpoint being evaluated with testing.py rather than a
  training run to be timed.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: 3D self-attention conditional GAN whose training time per epoch and time to
    plateau are measured
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline network timed under the same protocol for the cost comparison
- name: 3D conditional GAN
  role: Baseline network timed under the same protocol for the cost comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:bb9226af263188d69bd71fdeeee9ee6e857b6ce6ac234758435d4612cd8ed418
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# training-convergence-plateau-detection

## Summary

Instrument GAN training runs (e.g., SC-GAN vs. 2D cGAN, 3D cGAN, Att cGAN baselines) to record wall-clock minutes per epoch and cumulative minutes until the training loss plateaus, quantifying the computational burden reported for the paper's networks.

## When to use

Use when comparing the training wall-clock cost of generative models (minutes per epoch, time to plateau) or when you must document the computational burden of a 3D GAN before committing GPU resources.

## When NOT to use

- The goal is synthesis accuracy (normalized RMS error, peak SNR, structural similarity) rather than training-cost benchmarking.
- Only a single network is available, so no cross-model timing comparison (SC-GAN vs. Att cGAN, etc.) can be formed.
- Input is a pre-trained checkpoint being evaluated with testing.py rather than a training run to be timed.

## Inputs

- Training/val/test directories of subject-wise multimodal NIfTI volumes (modality_1.nii.gz, ..., target.nii.gz) per the SC-GAN README layout
- Training script configuration: --trainig_size, --gpu, --epoches, --img_size, --data_dir, --modalities, --logdir
- Hyperparameter overrides: --lr (0.001), --g_reg/--d_reg (0.001), --l1_weight (200), --B_weight (200)

## Outputs

- Wall-clock timing log per epoch for each network (SC-GAN, 2D cGAN, 3D cGAN, Att cGAN)
- time_cost_comparison.csv with one row per network and columns for minutes per epoch and minutes to plateau (SC-GAN ~65 min/epoch, ~1300 min to plateau)

## How to apply

Train SC-GAN and the comparison baselines (2D conditional GAN, 3D conditional GAN, Att cGAN) under identical settings using TensorFlow on the NVIDIA GPU cluster, with the SC-GAN hyperparameters (mu=200, gamma=200, lambda=20, regularizers 0.001, initial learning rate 0.001, default 120 epochs, img_size 256 as in the README). Instrument each run to log wall-clock time per epoch and cumulative time, and declare the plateau when the training loss stops improving (approximately 65 minutes per epoch and 1300 minutes to plateau for SC-GAN, matching Att cGAN). Aggregate per-model timings into a single table (time_cost_comparison.csv) with one row per network so the lengthy-training, GPU-memory-heavy limitation of SC-GAN can be verified quantitatively.

## Related tools

- **SC-GAN** (3D self-attention conditional GAN whose training time per epoch and time to plateau are measured) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline network timed under the same protocol for the cost comparison)
- **3D conditional GAN** (Baseline network timed under the same protocol for the cost comparison)

## Examples

```
python training.py --trainig_size= --gpu= --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --logdir=logs/scgan
```

## Evaluation signals

- time_cost_comparison.csv exists and contains exactly one row per network: SC-GAN, 2D cGAN, 3D cGAN, Att cGAN.
- SC-GAN's recorded values are consistent with the reported ~65 minutes per epoch and ~1300 minutes to reach the training plateau.
- All four networks are trained with the documented hyperparameters (lr=0.001, g_reg=d_reg=0.001, l1_weight=200, B_weight=200) so the timing comparison is apples-to-apples.
- Plateau declaration is justified by a flat training loss over consecutive epochs in the TensorBoard log (--logdir output).
- Timings reflect the paper's stated limitation that SC-GAN 'requires lengthy training and heavy computational resources, such as GPU memory.'

## Limitations

- Computational cost is the main limitation of SC-GAN: lengthy training and heavy GPU memory requirements, so runs may be infeasible on smaller GPUs.
- Plateau timing is approximate (~1300 minutes) and depends on cluster/GPU hardware, so absolute values may not transfer across setups.
- The README does not document timing instrumentation, so the plateau criterion must be defined and logged by the practitioner.

## Evidence

- [results] SC-GAN required 65 minutes per epoch and 1300 minutes to reach plateau, the same training time cost as the Att cGAN baseline.: [evidence span withheld — anchor status: not_found]
- [readme] Use the following command to run the training script:

python training.py --trainig_size= --gpu= --epoches=: "Use the following command to run the training script:

python training.py --trainig_size= --gpu= --epoches="
