---
name: gan-component-ablation-experiment-design
description: Use when a GAN-based synthesis model must be decomposed into architectural/loss
  components and each component's contribution to image-quality metrics needs to be
  ranked via controlled ablation.
when_to_use_negative:
- The goal is benchmarking SC-GAN against other architectures (2D conditional GAN,
  3D conditional GAN, 3D U-Net) rather than decomposing SC-GAN's own components —
  use the comparative-evaluation workflow instead
- Only a single trained model and limited compute are available — the paper notes
  computational cost (lengthy training, heavy GPU memory) is SC-GAN's main limitation,
  making five retraining runs infeasible
- The synthesis task involves quantitative modalities where 0–1 normalization distorts
  metric values and training stability differs from the PET/DTI tasks studied
edam_operation: http://edamontology.org/operation_3559
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3384
tools:
- name: SC-GAN
  role: Base 3D self-attention conditional GAN whose source code provides the training/testing
    scripts used to create and run each ablation variant
  repo: https://github.com/Haoyulance/SC-GAN
- name: TensorFlow
  role: Deep-learning framework in which the SC-GAN training and ablation variants
    are implemented (listed in repository requirements)
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:30fa9456213815b0bb7f6c69f03c5c033e84087a5e23f2c1bdaf9bf0a48be603
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3559
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# GAN component ablation experiment design

## Summary

A method for quantifying which components of a GAN (e.g., SC-GAN's self-attention, adversarial loss, B-rmse, spectral normalization, feature matching) drive synthesis performance by training one variant per removed component and ranking metric degradation on a held-out test set.

## When to use

Use when a GAN-based synthesis model must be decomposed into architectural/loss components and each component's contribution to image-quality metrics needs to be ranked via controlled ablation.

## When NOT to use

- The goal is benchmarking SC-GAN against other architectures (2D conditional GAN, 3D conditional GAN, 3D U-Net) rather than decomposing SC-GAN's own components — use the comparative-evaluation workflow instead
- Only a single trained model and limited compute are available — the paper notes computational cost (lengthy training, heavy GPU memory) is SC-GAN's main limitation, making five retraining runs infeasible
- The synthesis task involves quantitative modalities where 0–1 normalization distorts metric values and training stability differs from the PET/DTI tasks studied

## Inputs

- Training corpus of subject folders with input modality NIfTI files (e.g., T1w, FLAIR .nii.gz) and target modality (e.g., av45 PET target.nii.gz), split into train/val/test directories
- Reference SC-GAN training script and optimal hyperparameters (lr=0.001, l1_weight=200, B_weight=200, g_reg=0.001, d_reg=0.001, img_size=256)
- Ground-truth target images for the held-out test set (58 subjects for the ADNI-3 PET task)

## Outputs

- Five trained ablation model checkpoints (no self-attention, no adversarial learning, no B-rmse, no spectral normalization, no feature matching)
- Synthesized PET images per ablation variant for the test set
- ablation_results.csv tabulating NRMSE, peak SNR, and structural similarity for each ablation condition plus full SC-GAN
- Component-contribution ranking by metric degradation magnitude

## How to apply

Set up one ablation variant per removed component — for SC-GAN: (1) remove the self-attention module from the generator's down-/up-sampling paths and the discriminator, (2) remove the adversarial loss term, (3) remove the brain-area RMSE loss, (4) remove spectral normalization, and (5) remove the feature matching loss. Train each variant on the same task (e.g., ADNI-3 T1w+FLAIR to amyloid PET synthesis, 207 training subjects) using the full model's optimal hyperparameters: learning rate 0.001 with cosine decay, batch size 1, l1_weight 200, B_weight 200, generator/discriminator regularizers 0.001, image size 256, ~120 epochs per the README defaults. Run inference of each trained variant on the same test set (58 subjects) and compute normalized RMSE (NRMSE), peak SNR, and structural similarity against ground-truth images. Rank component importance by the magnitude of metric degradation when each component is removed, and verify that removing adversarial learning and self-attention produces the largest degradation, followed by spectral normalization and B-rmse. Compile NRMSE, peak SNR, and SSIM for all five ablations plus full SC-GAN into a results table (e.g., ablation_results.csv).

## Related tools

- **SC-GAN** (Base 3D self-attention conditional GAN whose source code provides the training/testing scripts used to create and run each ablation variant) — https://github.com/Haoyulance/SC-GAN
- **TensorFlow** (Deep-learning framework in which the SC-GAN training and ablation variants are implemented (listed in repository requirements)) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python training.py --training_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --lr=0.001 --l1_weight=200 --B_weight=200
```

## Evaluation signals

- Removing adversarial learning and the self-attention module yields the largest degradation in NRMSE, peak SNR, and structural similarity, matching the paper's ablation finding
- Spectral normalization and B-rmse removal produce the next-largest metric degradation, consistent with the reported ranking
- All five ablation variants plus full SC-GAN are trained with identical hyperparameters (lr 0.001, l1_weight 200, B_weight 200, img_size 256) and evaluated on the same 58-subject test set
- ablation_results.csv contains six rows (five ablations + full SC-GAN) with all three metrics populated and comparable ranges
- Synthesized outputs for every ablation are generated from the same test-subject inputs, so metric differences are attributable to the removed component only

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy GPU memory, so five ablation retrains are expensive
- Conclusions are grounded on the ADNI-3 PET synthesis task (207 train / 58 test subjects); rankings may differ for other modalities or datasets
- The in-house test suggesting no normalization dependency in SC-GAN is not presented, so normalization-related ablation conclusions cannot be fully verified

## Evidence
