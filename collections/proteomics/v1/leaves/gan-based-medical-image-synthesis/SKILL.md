---
name: gan-based-medical-image-synthesis
description: Use when the task is cross-domain neuroimage synthesis (e.g., T1w/FLAIR
  MRI to PET or DTI-derived maps) or multidimensional diffusion MRI superresolution
  where stable GAN training and 3D spatial consistency matter.
when_to_use_negative:
- Synthesizing quantitative modalities (e.g., DTI metrics) when preserving absolute
  quantitative values matters — the 0–1 min-max rescaling of target images negatively
  affects quantitative DTI metric values
- When training time or GPU memory is severely constrained — computational cost (lengthy
  training, heavy GPU memory) is the main limitation of SC-GAN
- Tasks requiring 2D slice-wise synthesis where 3D consistency is not needed — 2D
  conditional GAN is a lighter alternative (SC-GAN prediction error was 18–29% lower,
  but at higher compute cost)
edam_operation: http://edamontology.org/operation_3699
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3304
tools:
- name: SC-GAN
  role: Primary 3D self-attention conditional GAN architecture with spectral normalization
    and feature matching used for synthesis and MUDI superresolution
  repo: https://github.com/Haoyulance/SC-GAN
- name: 3D Unet
  role: Baseline regression backbone trained on identical data for fair comparison
    of superresolution performance
- name: 3D conditional GAN
  role: Baseline GAN without self-attention/spectral normalization for comparison
- name: FSL TOPUP
  role: Corrects eddy current distortion in diffusion MRI before synthesis
- name: FSL EDDY
  role: Corrects involuntary movement in diffusion MRI before synthesis
- name: ANTs (Advanced Normalization Tools)
  role: N4ITK field inhomogeneity correction of FLAIR and intermodal co-registration
    (antsIntermodalityIntrasubject)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:8aa3c19c966236c455c56473b94e574bfdbb80c44fcf879c62836b6cba43d47a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 3D Unet
  - 3D conditional GAN
  - ANTs (Advanced Normalization Tools)
  - FSL EDDY
  - FSL TOPUP
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3699
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3304
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# gan-based-medical-image-synthesis

## Summary

Train SC-GAN, a 3D self-attention conditional GAN with spectral normalization, feature matching, and brain-area RMSE loss, to synthesize or superresolve medical images (e.g., PET, FA/MD maps, or MUDI volumes), comparing against 3D Unet and 3D cGAN baselines.

## When to use

Use when the task is cross-domain neuroimage synthesis (e.g., T1w/FLAIR MRI to PET or DTI-derived maps) or multidimensional diffusion MRI superresolution where stable GAN training and 3D spatial consistency matter.

## When NOT to use

- Synthesizing quantitative modalities (e.g., DTI metrics) when preserving absolute quantitative values matters — the 0–1 min-max rescaling of target images negatively affects quantitative DTI metric values
- When training time or GPU memory is severely constrained — computational cost (lengthy training, heavy GPU memory) is the main limitation of SC-GAN
- Tasks requiring 2D slice-wise synthesis where 3D consistency is not needed — 2D conditional GAN is a lighter alternative (SC-GAN prediction error was 18–29% lower, but at higher compute cost)

## Inputs

- Multimodal MRI volumes in NIfTI format (.nii.gz), e.g., T1w and FLAIR from ADNI-3 (https://ida.loni.usc.edu/)
- MUDI challenge volumes of size 41 × 46 × 28 × 1344 (four training sets, one validation set)
- 6-element acquisition-protocol vectors (gradient-encoding directions x/y/z, b-value, TE, TI) for MUDI superresolution

## Outputs

- Synthesized target modality volumes (e.g., av45 PET, fractional anisotropy, mean diffusivity maps)
- Superresolved MUDI volumes at 82 × 92 × 56 target resolution
- Comparison table of NRMSE, PSNR, and SSIM metrics across models (e.g., mudi_superresolution_results.csv)

## How to apply

Preprocess inputs (NIfTI .nii.gz per subject/modality, e.g., ADNI-3 T1w/FLAIR) with eddy/motion correction (FSL TOPUP/EDDY), N4ITK bias correction, co-registration, Z-score normalization followed by min-max rescaling of voxels to 0–1. Organize data into train/val/test subject folders with modality_*.nii.gz files plus target.nii.gz, then train with the SC-GAN architecture (3D conditional GAN + self-attention + spectral normalization) using lr=0.001, batch size 4, ~120 epochs, l1_weight=200 and B_weight=200 for the brain-area RMSE loss. For MUDI superresolution, encode acquisition-protocol parameters (gradient directions x/y/z, b-value, TE, TI) as a 6-element vector injected into discriminator feature maps via AdaIN, first superresolve to the 82×92×56 target resolution, then train a second SC-GAN to refine the reconstruction. Train 3D Unet and 3D cGAN on identical data as fair baselines and evaluate with normalized RMSE, peak SNR, and structural similarity.

## Related tools

- **SC-GAN** (Primary 3D self-attention conditional GAN architecture with spectral normalization and feature matching used for synthesis and MUDI superresolution) — https://github.com/Haoyulance/SC-GAN
- **3D Unet** (Baseline regression backbone trained on identical data for fair comparison of superresolution performance)
- **3D conditional GAN** (Baseline GAN without self-attention/spectral normalization for comparison)
- **FSL TOPUP** (Corrects eddy current distortion in diffusion MRI before synthesis)
- **FSL EDDY** (Corrects involuntary movement in diffusion MRI before synthesis)
- **ANTs (Advanced Normalization Tools)** (N4ITK field inhomogeneity correction of FLAIR and intermodal co-registration (antsIntermodalityIntrasubject))

## Examples

```
python training.py --training_size= --gpu=0 --epoches=120 --img_size=256 --data_dir=data/train --modalities=flair_t1w_av45 --logdir=logs/scgan
```

## Evaluation signals

- SC-GAN should achieve lower normalized RMSE (and higher peak SNR and structural similarity) than 3D Unet and 3D cGAN; for MUDI superresolution, lowest average MSE of 2.15 vs 2.52 (Unet) and 2.41 (cGAN)
- For multimodal synthesis, prediction error should be 18%, 24%, and 29% lower than 2D conditional GAN for FA, PET, and MD tasks respectively
- Output volumes match expected image geometry (e.g., 82 × 92 × 56 for MUDI superresolution target; 256³ default img_size for ADNI synthesis)
- Training converges stably — spectral normalization plus feature matching should prevent mode collapse/divergent discriminator loss across ~120 epochs
- Validation on held-out subject folders (val/ split with target.nii.gz) reproduces the reported metric ordering across all three tasks and all evaluation metrics

## Limitations

- Computational cost is the main limitation of SC-GAN: lengthy training and heavy GPU memory requirements
- Normalizing target DTI images to 0–1 for stable training negatively affects the quantitative value of DTI metrics, so normalization should be avoided for synthesizing quantitative modalities
- For MUDI superresolution, the authors lacked additional acquisition protocol data required for further experiments and analyses
- The claimed normalization-independence of SC-GAN was not presented, so it cannot be verified; generalization via knowledge distillation remains unexplored future work

## Evidence
