---
name: nrmse-image-quality-evaluation
description: Use when a 3D image synthesis or superresolution model (e.g., GAN or
  Unet) has produced predicted volumes and co-registered ground-truth volumes are
  available, so prediction error must be quantified per model.
when_to_use_negative:
- No co-registered ground-truth target exists for the synthesized volume, so NRMSE/PSNR/SSIM
  cannot be computed
- The target is a quantitative modality (e.g., DTI metrics) where intensity normalization
  to 0-1 distorts quantitative values — the paper notes normalization 'negatively
  affects the quantitative value of the metrics' and should be avoided for quantitative
  synthesis
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SC-GAN
  role: Produces the synthesized 3D volumes whose NRMSE/PSNR/SSIM are computed; the
    reference implementation with training.py/testing.py scripts
  repo: https://github.com/Haoyulance/SC-GAN
- name: FreeSurfer 5.3.0
  role: Provides the T1w parcellation masks used to compute regional (WM/GM/CSF) NRMSE
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:71df897d1c2351af09c9545b8e42e0f8c08d578ff99b7e9249063e6ed08e18b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 3D Unet
  - FreeSurfer
  - FreeSurfer 5.3.0
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# nrmse-image-quality-evaluation

## Summary

Evaluation skill for image-synthesis pipelines that scores synthesized volumes against ground truth using normalized RMSE (whole-brain and regional), peak SNR, and structural similarity, as used to benchmark SC-GAN against 2D/3D conditional GANs on ADNI-3 PET/FA/MD synthesis and MUDI superresolution.

## When to use

Use when a 3D image synthesis or superresolution model (e.g., GAN or Unet) has produced predicted volumes and co-registered ground-truth volumes are available, so prediction error must be quantified per model.

## When NOT to use

- No co-registered ground-truth target exists for the synthesized volume, so NRMSE/PSNR/SSIM cannot be computed
- The target is a quantitative modality (e.g., DTI metrics) where intensity normalization to 0-1 distorts quantitative values — the paper notes normalization 'negatively affects the quantitative value of the metrics' and should be avoided for quantitative synthesis

## Inputs

- Synthesized 3D volumes (e.g., FA, MD, or PET NIfTI outputs from testing.py)
- Ground-truth 3D volumes co-registered to the same space (e.g., ADNI-3 FA maps, PET SUVR images)
- Anatomical parcellation masks (FreeSurfer 5.3.0 labels for WM, GM, CSF derived from T1w)
- For MUDI: superresolved validation-set volumes (82 x 92 x 56) versus ground-truth multidimensional diffusion volumes

## Outputs

- Whole-brain NRMSE, peak SNR, and structural similarity per model (mean/SD across test subjects)
- Regional NRMSE table for white matter, gray matter, and CSF (e.g., fa_synthesis_nrmse_table.csv)
- MUDI superresolution comparison table of NRMSE/PSNR/SSIM across models (mudi_superresolution_results.csv)

## How to apply

Compare each synthesized volume against its ground-truth volume to compute three whole-brain metrics: normalized RMS error, peak SNR, and structural similarity. For 2D networks that emit slices, evaluate the 3D output directly so scores are comparable across architectures. Additionally compute regional NRMSE within white matter, gray matter, and CSF using anatomical parcellation masks derived from the T1w preprocessing (FreeSurfer 5.3.0 in this study). Report NRMSE as mean/SD across test subjects (e.g., SC-GAN FA whole-brain 0.078/0.012; WM 0.088/0.012, GM 0.062/0.01, CSF 0.073/0.014) and interpret relative model rankings, e.g., SC-GAN's FA prediction error was 18% lower than 2D conditional GAN. On the MUDI superresolution task, the same NRMSE/PSNR/SSIM trio is computed on the validation subject to rank SC-GAN (MSE 2.15) over 3D cGAN (2.41) and 3D Unet (2.52).

## Related tools

- **SC-GAN** (Produces the synthesized 3D volumes whose NRMSE/PSNR/SSIM are computed; the reference implementation with training.py/testing.py scripts) — https://github.com/Haoyulance/SC-GAN
- **FreeSurfer 5.3.0** (Provides the T1w parcellation masks used to compute regional (WM/GM/CSF) NRMSE)

## Examples

```
python testing.py --test_size= --gpu= --img_size= --data_dir= --modalities= --logdir= --output=
```

## Evaluation signals

- Whole-brain FA NRMSE for SC-GAN should fall near the reported 0.078 (SD 0.012) with regional values WM ~0.088, GM ~0.062, CSF ~0.073
- SC-GAN must show lower NRMSE (and higher peak SNR/SSIM) than 2D cGAN, 3D cGAN, and Att cGAN baselines; FA error ~18% lower than 2D cGAN is the acceptance criterion
- On MUDI, SC-GAN average MSE should be lowest (~2.15) versus 3D cGAN (~2.41) and 3D Unet (~2.52) on the validation subject
- The 2D network's output must be evaluated as a 3D volume so metrics are comparable across architectures
- Regional NRMSE must be computed with FreeSurfer 5.3.0 parcellation masks from the T1w preprocessing, not ad-hoc ROIs

## Limitations

- Computational cost is the main limitation of SC-GAN; lengthy training and heavy GPU memory requirements make large-scale re-evaluation expensive
- Normalizing target DTI images to 0-1 for stable training negatively affects the quantitative value of DTI metrics, so NRMSE-based comparisons on normalized outputs lose absolute quantitative meaning
- For the MUDI superresolution application, additional acquisition protocol data were lacking, limiting further experiments and analyses

## Evidence
