---
name: self-attention-module-contribution-analysis
description: Use when a trained conditional GAN for image synthesis must be attributed
  to specific architectural or loss components, e.g. testing whether a self-attention
  module improves 3D GAN consistency.
when_to_use_negative:
- The goal is a single best synthesis model rather than component attribution — compare
  full SC-GAN against baselines (2D conditional GAN, 3D conditional GAN, 3D Unet)
  instead of running ablations
- The synthesis task is not a 3D conditional GAN with self-attention components to
  remove (e.g. pure 2D GANs or U-Net regression lack the adversarial/spectral-norm/feature-matching
  terms being ablated)
- Quantitative-value targets (e.g. quantitative DTI metrics) where the authors warn
  that min-max normalization to 0-1 should be avoided — the ablation hyperparameters
  here assume the PET-style normalized training regime
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: The 3D self-attention conditional GAN framework whose training and testing
    scripts are run per ablation variant with components selectively removed
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Deep-learning backend in which the SC-GAN training script and its ablation
    variants are implemented and executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:1b245a5a4314e06fb8025e30238725ef442e53c1bb619925712b6432b190274b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# self-attention-module-contribution-analysis

## Summary

A five-condition ablation protocol for SC-GAN that isolates which components (self-attention module, adversarial learning, B-rmse loss, spectral normalization, feature matching) drive synthesis quality on 3D multimodal neuroimaging tasks, quantified by NRMSE, peak SNR, and structural similarity.

## When to use

Use when a trained conditional GAN for image synthesis must be attributed to specific architectural or loss components, e.g. testing whether a self-attention module improves 3D GAN consistency.

## When NOT to use

- The goal is a single best synthesis model rather than component attribution — compare full SC-GAN against baselines (2D conditional GAN, 3D conditional GAN, 3D Unet) instead of running ablations
- The synthesis task is not a 3D conditional GAN with self-attention components to remove (e.g. pure 2D GANs or U-Net regression lack the adversarial/spectral-norm/feature-matching terms being ablated)
- Quantitative-value targets (e.g. quantitative DTI metrics) where the authors warn that min-max normalization to 0-1 should be avoided — the ablation hyperparameters here assume the PET-style normalized training regime

## Inputs

- ADNI-3 training subjects (207) with T1w + FLAIR NIfTI volumes and ground-truth amyloid PET (av45) target per subject, organized as data/train/<subject>/{modality_1.nii.gz, ..., target.nii.gz}
- ADNI-3 test set (58 subjects) with T1w + FLAIR inputs and ground-truth PET
- Trained SC-GAN checkpoints from the full model and each of the five ablation variants

## Outputs

- Synthesized PET volumes per ablation variant for the 58-subject test set
- ablation_results.csv with NRMSE, peak SNR, and structural similarity for the full SC-GAN and all five ablation conditions
- Ranked contribution ordering of components (adversarial learning and self-attention first, then spectral normalization, then B-rmse, then feature matching)

## How to apply

Set up five ablation variants of SC-GAN by selectively removing: (1) the self-attention module from the generator's down/up-sampling paths and discriminator, (2) the adversarial loss term, (3) the brain-area RMSE (B-rmse) loss, (4) spectral normalization, and (5) the feature matching loss. Train each variant on the ADNI-3 PET synthesis task (207 training subjects, T1w + FLAIR inputs targeting amyloid PET) with the SC-GAN training script using optimal hyperparameters: learning rate 0.001 with cosine decay, 120 epochs, batch size 1, img_size 256, l1_weight 200, B_weight 200, and generator/discriminator regularizers 0.001. Run inference with each trained model on the 58-subject test set and compute normalized RMSE, peak SNR, and structural similarity against ground-truth PET. Rank component contributions by the magnitude of metric degradation when each component is removed, and compile results into ablation_results.csv. The expected validation outcome is that removing adversarial learning and the self-attention module causes the largest degradation, followed by spectral normalization and B-rmse.

## Related tools

- **SC-GAN** (The 3D self-attention conditional GAN framework whose training and testing scripts are run per ablation variant with components selectively removed) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Deep-learning backend in which the SC-GAN training script and its ablation variants are implemented and executed)

## Examples

```
python training.py --training_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --logdir=logs/scgan_full
```

## Evaluation signals

- Removing adversarial learning and the self-attention module produces the largest degradation in NRMSE, peak SNR, and structural similarity, matching the paper's reported ablation finding
- The remaining components rank as spectral normalization, then B-rmse, then feature matching by metric degradation magnitude
- Full SC-GAN (no component removed) achieves the best NRMSE, peak SNR, and structural similarity among all six conditions
- All six conditions were trained with identical hyperparameters (lr 0.001, l1_weight 200, B_weight 200, img_size 256, 120 epochs) so metric differences are attributable to the removed component
- ablation_results.csv contains exactly five ablation conditions plus full SC-GAN across all three evaluation metrics

## Limitations

- Computational cost is the main limitation of SC-GAN; training each ablation variant requires lengthy training and heavy GPU memory
- The ablation was demonstrated on the ADNI-3 PET synthesis task; transfer of the contribution ranking to other targets (FA, MD, MUDI superresolution) is not established here
- Training stability depends on the normalization regime; for synthesizing quantitative modalities the authors warn the normalization step should be avoided

## Evidence
