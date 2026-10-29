---
name: 3d-conditional-gan-training
description: Use when the goal is volumetric (256x256x256) cross-modality neuroimage
  synthesis — e.g., predicting PET, fractional anisotropy, or mean diffusivity maps
  from T1w/FLAIR MRI — and stable 3D GAN training plus baseline comparison on NRMSE/PSNR/SSIM
  is required.
when_to_use_negative:
- The task is single-slice/2D image synthesis or the inputs are not co-registered
  volumetric MRI/PRI pairs at ~256^3 resolution — the 3D GAN's benefit is volumetric
  consistency
- 'Computational resources are limited: SC-GAN requires lengthy training and heavy
  GPU memory, so a 3D Unet or 2D network may be the practical choice'
- The target is a quantitative modality whose absolute values matter (e.g., DTI metrics)
  and min-max rescaling to 0-1 would distort the quantitative values — the authors
  advise avoiding the normalization step for such targets
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3384
tools:
- name: SC-GAN
  role: The 3D self-attention conditional GAN being trained; TensorFlow implementation
    with spectral normalization, feature matching, and B-rmse losses
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline network for comparison; its 3D output is evaluated directly on the
    held-out test set
- name: 3D conditional GAN
  role: Baseline network that SC-GAN extends with the self-attention module and stabilization
    techniques
- name: Attention cGAN (Att cGAN)
  role: Attention-equipped baseline trained with mu=200, nu_G=nu_D=0.001, batch size
    1, lr=0.001
- name: 3D Unet
  role: Non-GAN baseline against which SC-GAN's synthesis is compared
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:409de91103b090c855b4d66f8b626f01b22a77ea4701b6033cd5ce57ff0f1347
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D Unet
  - 3D conditional GAN
  - Attention cGAN (Att cGAN)
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# 3d-conditional-gan-training

## Summary

Train SC-GAN, a 3D self-attention conditional GAN with spectral normalization, feature matching, and brain-area RMSE losses, to synthesize a target neuroimaging modality (e.g., amyloid PET) from multimodal MRI inputs, and evaluate it against 2D/3D cGAN and Att cGAN baselines using NRMSE, peak SNR, and SSIM.

## When to use

Use when the goal is volumetric (256x256x256) cross-modality neuroimage synthesis — e.g., predicting PET, fractional anisotropy, or mean diffusivity maps from T1w/FLAIR MRI — and stable 3D GAN training plus baseline comparison on NRMSE/PSNR/SSIM is required.

## When NOT to use

- The task is single-slice/2D image synthesis or the inputs are not co-registered volumetric MRI/PRI pairs at ~256^3 resolution — the 3D GAN's benefit is volumetric consistency
- Computational resources are limited: SC-GAN requires lengthy training and heavy GPU memory, so a 3D Unet or 2D network may be the practical choice
- The target is a quantitative modality whose absolute values matter (e.g., DTI metrics) and min-max rescaling to 0-1 would distort the quantitative values — the authors advise avoiding the normalization step for such targets

## Inputs

- ADNI-3 subject-level NIfTI volumes (.nii.gz): T1w and FLAIR MRI inputs plus amyloid PET targets (265 subjects: 207 train / 58 test)
- Data directory structured as train/val/test per-subject folders with modality_1.nii.gz, modality_2.nii.gz, ..., target.nii.gz
- Hyperparameter configuration: lr=0.001 (cosine decay), l1_weight=200, B_weight=200, g_reg=d_reg=0.001, img_size=256, epochs=120

## Outputs

- Synthesized 3D target volumes (e.g., PET) for each held-out test subject
- Per-network NRMSE, peak SNR, and SSIM comparison table (pet_synthesis_comparison_table.csv)
- Ablation results (ablation_results.csv) with NRMSE, PSNR, SSIM for the five ablation variants plus full SC-GAN

## How to apply

Organize data as train/val/test subject directories of NIfTI (.nii.gz) volumes, where each subject holds input modality files (e.g., flair_t1w) and a target.nii.gz; preprocess with Z-score normalization followed by min-max rescaling of voxel intensities to 0-1. Train SC-GAN with the repository's training.py at img_size=256, batch size 1, learning rate 0.001 with cosine decay, l1_weight (mu)=200, B_weight (gamma)=200, and generator/discriminator regularizers 0.001, using 5-fold cross-validation during hyperparameter tuning; train 2D cGAN, 3D cGAN, and Att cGAN baselines with their tuned settings for a fair comparison. Run testing.py on held-out subjects (e.g., the 58-subject ADNI-3 test set) and compute normalized RMSE, peak SNR, and structural similarity of synthesized PET against ground truth. Optionally run five ablations (removing self-attention, adversarial learning, B-rmse, spectral normalization, or feature matching) and rank component contributions by metric degradation. Correct application is confirmed when SC-GAN shows the lowest NRMSE (about 24% lower error than 2D cGAN on PET synthesis) and best rank on all three metrics.

## Related tools

- **SC-GAN** (The 3D self-attention conditional GAN being trained; TensorFlow implementation with spectral normalization, feature matching, and B-rmse losses) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline network for comparison; its 3D output is evaluated directly on the held-out test set)
- **3D conditional GAN** (Baseline network that SC-GAN extends with the self-attention module and stabilization techniques)
- **Attention cGAN (Att cGAN)** (Attention-equipped baseline trained with mu=200, nu_G=nu_D=0.001, batch size 1, lr=0.001)
- **3D Unet** (Non-GAN baseline against which SC-GAN's synthesis is compared)

## Examples

```
python training.py --training_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=./data --modalities=flair_t1w_av45 --logdir=./logs --lr=0.001 --l1_weight=200 --B_weight=200
```

## Evaluation signals

- SC-GAN achieves the lowest NRMSE and the highest peak SNR and SSIM among all compared networks (2D cGAN, 3D cGAN, Att cGAN)
- SC-GAN's PET synthesis prediction error is approximately 24% lower than the 2D conditional GAN's
- Ablation ranking matches expectation: removing adversarial learning and the self-attention module produces the largest metric degradation, followed by B-rmse and spectral normalization
- Training converges stably (spectral normalization and feature matching are in place) at the documented optimal hyperparameters mu=200, gamma=200, lr=0.001 with cosine decay
- Output volumes are 256x256x256, matching the img_size=256 configuration and the input MRI resolution

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory
- Min-max rescaling of target intensities to 0-1 stabilizes training but negatively affects the quantitative value of DTI metrics; normalization should be avoided when synthesizing quantitative modalities
- The in-house test suggesting no normalization dependency in SC-GAN was not presented, so it cannot be verified

## Evidence

- [abstract] SC-GAN outperformed other state-of-the-art GAN networks including 3D conditional GAN in all three tasks across all evaluation metrics: "In general, SC-GAN outperformed other state-of-the-art GAN networks including 3D conditional GAN in all three tasks across all evaluation metrics"
- [abstract] Prediction error 24% lower than 2D cGAN on PET: "Prediction error of the SC-GAN was 18%, 24% and 29% lower compared to 2D conditional GAN for fractional anisotropy, PET and mean diffusivity tasks, respectively"
- [methods] Z-score then min-max normalization of voxel intensities: "We implemented Z-score normalization for all four tasks, then applied min-max rescaling to scale the voxels’ intensity from between 0 to 1 before the training"
- [abstract] Ablation major contributors: "The ablation experiment showed that the major contributors to the improved performance of SC-GAN are the adversarial learning and the self-attention module, followed by the spectral normalization"
- [readme] Training script CLI from README: "python training.py --trainig_size= --gpu= --epoches= --img_size= --data_dir= --modalities= --logdir="
