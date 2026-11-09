---
name: pet-image-synthesis-from-mri
description: Use when a task requires predicting 3D PET images (e.g., av45 amyloid
  PET) from paired T1w/FLAIR MRI volumes at 256x256x256 resolution, with quantitative
  image-quality benchmarking.
when_to_use_negative:
- The target modality is a quantitative map (e.g., DTI-derived FA or MD metrics) where
  normalizing voxel intensities to 0-1 would destroy the quantitative value of the
  metrics — the paper notes normalization negatively affects quantitative metrics
  and should be avoided for synthesizing quantitative modalities
- The input images are low-resolution or 2D slices and an end-to-end 3D pipeline at
  256x256x256 is not feasible — SC-GAN requires heavy GPU memory and lengthy training,
  which is its main limitation
- The task is superresolution of diffusion MRI with different acquisition protocols
  rather than cross-modal PET synthesis from T1w/FLAIR MRI
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3334
tools:
- name: SC-GAN
  role: Primary 3D self-attention conditional GAN with spectral normalization, feature
    matching, and brain-area RMSE loss used to synthesize PET from multimodal MRI
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline comparison network whose 3D output is evaluated directly; SC-GAN's
    PET prediction error is 24% lower
- name: 3D conditional GAN
  role: Baseline comparison network without self-attention or spectral normalization
- name: Attention cGAN (Att cGAN)
  role: Baseline comparison network with attention but trained in the comparison set
- name: TensorFlow
  role: Framework in which SC-GAN and the baselines are implemented and trained
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:e9884eacdd2383fa09383f3ad90297321a57d30aaf47598bd4ca358927570ff3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - Attention cGAN (Att cGAN)
  - SC-GAN
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# pet-image-synthesis-from-mri

## Summary

Synthesize 3D amyloid PET images from multimodal MRI (T1w and FLAIR) using SC-GAN, a 3D self-attention conditional GAN with spectral normalization, trained on the ADNI-3 cohort and benchmarked against 2D cGAN, 3D cGAN, and Att cGAN baselines using NRMSE, peak SNR, and SSIM.

## When to use

Use when a task requires predicting 3D PET images (e.g., av45 amyloid PET) from paired T1w/FLAIR MRI volumes at 256x256x256 resolution, with quantitative image-quality benchmarking.

## When NOT to use

- The target modality is a quantitative map (e.g., DTI-derived FA or MD metrics) where normalizing voxel intensities to 0-1 would destroy the quantitative value of the metrics — the paper notes normalization negatively affects quantitative metrics and should be avoided for synthesizing quantitative modalities
- The input images are low-resolution or 2D slices and an end-to-end 3D pipeline at 256x256x256 is not feasible — SC-GAN requires heavy GPU memory and lengthy training, which is its main limitation
- The task is superresolution of diffusion MRI with different acquisition protocols rather than cross-modal PET synthesis from T1w/FLAIR MRI

## Inputs

- Paired T1w and FLAIR MRI volumes (NIfTI, e.g., .nii.gz) per subject, organized as data/train, data/val, data/test subject directories
- Co-registered amyloid (av45) PET target volumes (NIfTI), upsampled to 256x256x256
- Subject split manifest: 207 ADNI-3 training subjects and 58 held-out testing subjects

## Outputs

- Synthesized 3D PET volumes (NIfTI) for each of the 58 test subjects
- Per-network comparison table (e.g., pet_synthesis_comparison_table.csv) with NRMSE, peak SNR, and SSIM for SC-GAN, 2D cGAN, 3D cGAN, and Att cGAN
- TensorBoard training logs saved to the --logdir directory

## How to apply

Load the ADNI-3 PET synthesis data set (265 subjects split into 207 training and 58 testing), upsample amyloid PET to 256x256x256, and preprocess all images with Z-score normalization followed by min-max rescaling of voxel intensities to 0-1 before training. Train SC-GAN in TensorFlow 3 with optimal hyperparameters mu=200, gamma=200, lambda=20, nu_G=0.001, nu_D=0.001, batch size=1, and initial learning rate 0.001 with cosine decay; train the 2D cGAN, 3D cGAN, and Att cGAN baselines with their tuned hyperparameters, using 5-fold cross-validation during hyperparameter tuning for all four networks. Run inference with each trained network on the 58 held-out testing subjects, evaluating the 3D output of the 2D network directly for comparability. Compute normalized RMSE (NRMSE), peak SNR, and structural similarity (SSIM) for each network's synthesized PET against ground truth and assemble a per-network comparison table. Correct application is validated when SC-GAN achieves the lowest NRMSE (approximately 24% lower prediction error than the 2D conditional GAN) and ranks best on all three metrics.

## Related tools

- **SC-GAN** (Primary 3D self-attention conditional GAN with spectral normalization, feature matching, and brain-area RMSE loss used to synthesize PET from multimodal MRI) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline comparison network whose 3D output is evaluated directly; SC-GAN's PET prediction error is 24% lower)
- **3D conditional GAN** (Baseline comparison network without self-attention or spectral normalization)
- **Attention cGAN (Att cGAN)** (Baseline comparison network with attention but trained in the comparison set)
- **TensorFlow** (Framework in which SC-GAN and the baselines are implemented and trained)

## Examples

```
python training.py --trainig_size=207 --gpu=0 --epoches=120 --img_size=256 --data_dir=data --modalities=flair_t1w_av45 --logdir=logs
```

## Evaluation signals

- SC-GAN attains the lowest NRMSE and the highest peak SNR and structural similarity (SSIM) among the four compared networks on the 58 test subjects
- SC-GAN's NRMSE is approximately 24% lower than the 2D conditional GAN for the PET task
- All input volumes are 256x256x256 with Z-score normalization followed by min-max rescaling of voxel intensities to the 0-1 range before training
- Hyperparameters match the tuned values: SC-GAN mu=200, gamma=200, lambda=20, nu_G=nu_D=0.001, batch size=1, initial lr=0.001 with cosine decay
- Synthesized PET volumes align with ground truth after the same preprocessing (PET co-registered on T1w MRI with T1w-derived brain mask applied)

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy computational resources, such as GPU memory
- Normalizing target images to 0-1 for stable training negatively affects the quantitative value of the metrics, so the normalization step should be avoided when synthesizing quantitative modalities
- Generalization of SC-GAN (e.g., via knowledge distillation) is unexplored and deferred to future research

## Evidence

- [methods] We implemented Z-score normalization for all four tasks, then applied min-max rescaling to scale the voxels' intensity from between 0 to 1 before the training: [evidence span withheld — anchor status: not_found]
