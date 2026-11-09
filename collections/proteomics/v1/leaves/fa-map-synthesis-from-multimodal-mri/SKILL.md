---
name: fa-map-synthesis-from-multimodal-mri
description: Use when the goal is to predict DTI-derived FA volumes from structural
  multimodal MRI (T1w + FLAIR) with a 3D GAN, and when quantitative image-quality
  metrics (NRMSE, PSNR, SSIM) against ground-truth FA are the expected output.
when_to_use_negative:
- When the target modality is a quantitative map whose absolute values matter (e.g.,
  MD synthesis) and the 0–1 normalization step would distort the quantitative value
  of the metrics — the paper notes normalization 'negatively affects the quantitative
  value of the metrics'
- When no ground-truth FA maps or paired T1w/FLAIR data exist, since the conditional
  GAN requires paired multimodal input and target volumes for training and evaluation
- When GPU memory or training time is severely constrained — SC-GAN 'requires lengthy
  training and heavy computational resources, such as GPU memory'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3304
- http://edamontology.org/topic_3474
tools:
- name: SC-GAN
  role: Primary 3D self-attention conditional GAN (with spectral normalization and
    feature matching) used to synthesize FA maps from T1w + FLAIR inputs
  repo: https://github.com/Haoyulance/SC-GAN
- name: 2D conditional GAN
  role: Baseline GAN (mu=100, nuG=nuD=0.01, batch size=4, lr=0.0002) against which
    SC-GAN's 18% lower FA prediction error is benchmarked
- name: 3D conditional GAN
  role: Baseline 3D GAN (mu=200, nuG=nuD=0.001, batch size=1, lr=0.002) used for comparison
    on the FA task
- name: FreeSurfer 5.3.0
  role: T1w preprocessing and parcellation, supplying regional masks (WM, GM, CSF)
    for regional NRMSE computation
- name: FSL EDDY
  role: Corrects eddy current distortion and involuntary movement in diffusion MRI
    before DTI fitting for ground-truth FA
- name: N4ITK module
  role: Corrects nonuniform field inhomogeneity in FLAIR images (via ANTs) prior to
    registration and synthesis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:4cf2c8d76d432b42b074b4f71a4c51729e9f2e5bb72f3b5da822981e2a41062e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - 2D conditional GAN
  - 3D conditional GAN
  - FSL EDDY
  - FreeSurfer
  - FreeSurfer 5.3.0
  - N4ITK module
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3304
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# fa-map-synthesis-from-multimodal-mri

## Summary

Synthesize fractional anisotropy (FA) maps from co-registered T1w and FLAIR MRI using the 3D self-attention conditional GAN (SC-GAN) with spectral normalization, on ADNI-3 data (497 subjects: 398 training / 99 testing), achieving whole-brain NRMSE of 0.078/0.012 and 18% lower prediction error than a 2D conditional GAN.

## When to use

Use when the goal is to predict DTI-derived FA volumes from structural multimodal MRI (T1w + FLAIR) with a 3D GAN, and when quantitative image-quality metrics (NRMSE, PSNR, SSIM) against ground-truth FA are the expected output.

## When NOT to use

- When the target modality is a quantitative map whose absolute values matter (e.g., MD synthesis) and the 0–1 normalization step would distort the quantitative value of the metrics — the paper notes normalization 'negatively affects the quantitative value of the metrics'
- When no ground-truth FA maps or paired T1w/FLAIR data exist, since the conditional GAN requires paired multimodal input and target volumes for training and evaluation
- When GPU memory or training time is severely constrained — SC-GAN 'requires lengthy training and heavy computational resources, such as GPU memory'

## Inputs

- Preprocessed, co-registered T1w and FLAIR NIfTI volumes (.nii.gz) for each subject
- Ground-truth FA maps fitted from diffusion MRI (FSL TOPUP/EDDY-corrected, DTI fit via Quantitative Imaging Toolkit)
- FreeSurfer 5.3.0 parcellation masks derived from T1w preprocessing (for regional NRMSE)
- ADNI-3 subject data (497 subjects: 398 train / 99 test) obtained via https://ida.loni.usc.edu/

## Outputs

- Synthesized FA volumes for each of the 99 test subjects (one per network: SC-GAN, 2D cGAN, 3D cGAN, Att cGAN)
- Whole-brain image-quality metrics (NRMSE, PSNR, SSIM) per network versus ground-truth FA
- Regional NRMSE within white matter, gray matter, and CSF per network (e.g., fa_synthesis_nrmse_table.csv)

## How to apply

Preprocess the T1w (FreeSurfer 5.3.0 for preprocessing/parcellation) and FLAIR (N4ITK inhomogeneity correction via ANTs, then co-registration to T1w with antsIntermodalityIntrasubject) images, correct diffusion data with FSL TOPUP/EDDY, fit the DTI with the Quantitative Imaging Toolkit, then apply Z-score normalization followed by min-max rescaling of voxel intensities to 0–1 before training. Train SC-GAN in TensorFlow 1.12.2 with the selected hyperparameters mu=200, gamma=200, lambda=20, nuG=0.001, nuD=0.001, batch size=1, initial learning rate 0.001 with cosine decay (chosen by 5-fold cross-validation), using inputs at 256×256×256. Compare against baselines under their reported settings — 2D cGAN (mu=100, nuG=nuD=0.01, batch size=4, lr=0.0002), 3D cGAN (mu=200, nuG=nuD=0.001, batch size=1, lr=0.002), Att cGAN (mu=200, nuG=nuD=0.001, batch size=1, lr=0.001) — and run the testing script on the 99 held-out ADNI-3 subjects. Evaluate with normalized RMS error (whole-brain: 0.078/0.012; WM 0.088/0.012, GM 0.062/0.010, CSF 0.073/0.014 using FreeSurfer parcellation masks), peak SNR, and structural similarity; acceptance requires SC-GAN to yield lower NRMSE than all baselines, with ~18% lower error than the 2D conditional GAN for the FA task.

## Related tools

- **SC-GAN** (Primary 3D self-attention conditional GAN (with spectral normalization and feature matching) used to synthesize FA maps from T1w + FLAIR inputs) — https://github.com/Haoyulance/SC-GAN
- **2D conditional GAN** (Baseline GAN (mu=100, nuG=nuD=0.01, batch size=4, lr=0.0002) against which SC-GAN's 18% lower FA prediction error is benchmarked)
- **3D conditional GAN** (Baseline 3D GAN (mu=200, nuG=nuD=0.001, batch size=1, lr=0.002) used for comparison on the FA task)
- **FreeSurfer 5.3.0** (T1w preprocessing and parcellation, supplying regional masks (WM, GM, CSF) for regional NRMSE computation)
- **FSL EDDY** (Corrects eddy current distortion and involuntary movement in diffusion MRI before DTI fitting for ground-truth FA)
- **N4ITK module** (Corrects nonuniform field inhomogeneity in FLAIR images (via ANTs) prior to registration and synthesis)

## Examples

```
python testing.py --test_size=99 --gpu=0 --img_size=256 --data_dir=data/test --modalities=flair_t1w_fa --logdir=logs/fa --output=outputs/fa_scgan
```

## Evaluation signals

- Whole-brain NRMSE (mean/SD) of SC-GAN on the FA task should be approximately 0.078/0.012, with regional NRMSE of 0.088/0.012 (WM), 0.062/0.010 (GM), and 0.073/0.014 (CSF)
- SC-GAN prediction error must be lower than all baselines (2D cGAN, 3D cGAN, Att cGAN); per the paper it was 18% lower than the 2D conditional GAN for fractional anisotropy
- Hyperparameters should match the 5-fold cross-validation selection: mu=200, gamma=200, lambda=20, nuG=0.001, nuD=0.001, batch size=1, initial lr 0.001 with cosine decay
- All metrics computed on the same 99 held-out test subjects, with Z-score + min-max (0–1) normalization applied identically across networks for comparability
- Regional NRMSE must be computed within FreeSurfer 5.3.0 parcellation masks derived from the T1w preprocessing

## Limitations

- Computational cost is the main limitation of SC-GAN; it requires lengthy training and heavy GPU memory resources
- Normalizing target DTI images to 0–1 for stable training negatively affects the quantitative value of the DTI metrics; the paper suggests avoiding normalization when synthesizing quantitative modalities
- Hyperparameters (e.g., epochs default 120, image size 256) are dataset-specific to ADNI-3 at 256×256×256 and were selected via 5-fold cross-validation, so may need re-tuning on other cohorts

## Evidence
