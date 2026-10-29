---
name: adni-neuroimaging-data-handling
description: Use when raw ADNI-3 MRI/PET downloads must be turned into analysis-ready
  256x256x256 training/testing volumes for cross-modality synthesis (e.g., MRI-to-PET
  or MRI-to-FA/MD) with quality metrics such as NRMSE, peak SNR, and SSIM.
when_to_use_negative:
- Synthesizing quantitative modalities (e.g., DTI metrics) where min-max 0-1 rescaling
  would destroy the quantitative value of the metrics — the authors state the normalization
  step should be avoided in that case
- MUDI superresolution experiments requiring additional acquisition-protocol data
  that is not available in the ADNI release
- Data are already co-registered, intensity-normalized 256-cube volumes matching the
  README data/train|val|test structure
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: ANTs (Advanced Normalization Tools)
  role: N4ITK field-inhomogeneity correction of FLAIR and antsIntermodalityIntrasubject
    co-registration of FLAIR to T1w
- name: FSL TOPUP
  role: Eddy-current distortion and involuntary movement correction of diffusion MRI
- name: FSL EDDY
  role: Eddy-current distortion and involuntary movement correction of diffusion MRI
- name: LONI pipeline (Laboratory of Neuro Imaging pipeline)
  role: System used to conduct the data processing workflow
  repo: http://pipeline.loni.usc.edu
- name: FreeSurfer 5.3.0
  role: T1w preprocessing and parcellation producing the brain mask applied to PET
- name: SC-GAN
  role: Downstream consumer of the preprocessed ADNI-3 volumes (256-cube, 0-1 rescaled,
    207/58 split)
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:98f257a7fb0120369c4782838e92ed222ebd48b1303d20f360100fc2f96be041
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ANTs (Advanced Normalization Tools)
  - FSL EDDY
  - FSL TOPUP
  - FreeSurfer 5.3.0
  - LONI pipeline (Laboratory of Neuro Imaging pipeline)
  - SC-GAN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# adni-neuroimaging-data-handling

## Summary

End-to-end preparation of ADNI-3 multimodal neuroimaging data (T1w, FLAIR MRI, amyloid PET, dMRI) for 3D image-synthesis experiments: co-registration, denoising, Z-score + min-max normalization to 256x256x256 volumes, and 207/58 train/test splitting.

## When to use

Use when raw ADNI-3 MRI/PET downloads must be turned into analysis-ready 256x256x256 training/testing volumes for cross-modality synthesis (e.g., MRI-to-PET or MRI-to-FA/MD) with quality metrics such as NRMSE, peak SNR, and SSIM.

## When NOT to use

- Synthesizing quantitative modalities (e.g., DTI metrics) where min-max 0-1 rescaling would destroy the quantitative value of the metrics — the authors state the normalization step should be avoided in that case
- MUDI superresolution experiments requiring additional acquisition-protocol data that is not available in the ADNI release
- Data are already co-registered, intensity-normalized 256-cube volumes matching the README data/train|val|test structure

## Inputs

- ADNI-3 subject downloads (T1w, FLAIR MRI, amyloid PET, diffusion MRI) via https://ida.loni.usc.edu/
- NIfTI volumes (.nii.gz) organized as data/train|val|test/<subject>/modality_1.nii.gz, ..., target.nii.gz
- T1w-derived brain mask for PET masking
- PET frame series per subject (co-registered and combined)

## Outputs

- Preprocessed 256x256x256 Z-score normalized, min-max rescaled (0-1) multimodal volumes (207 train / 58 test subjects)
- Co-registered PET-on-T1w images with brain mask applied and SUVR-based quantitative measurements
- Corrected dMRI (TOPUP/EDDY) and fitted DTI maps (FA, MD)
- README-conformant directory tree consumable by training.py/testing.py

## How to apply

Obtain ADNI-3 data from the ADNI database (ida.loni.usc.edu) and organize it per the SC-GAN README into data/train, data/val, and data/test subject folders, each holding input modality NIfTI files (e.g., T1w, FLAIR) plus a target.nii.gz, with the 265 subjects split into 207 training and 58 testing. Correct FLAIR field inhomogeneity with the N4ITK module of ANTs and co-register FLAIR to T1w using antsIntermodalityIntrasubject; correct diffusion MRI for eddy currents and motion with FSL TOPUP/EDDY and fit DTI with the Quantitative Imaging Toolkit; co-register each PET frame to the first frame, combine frames into a single image to reduce motion artifacts, then co-register PET onto T1w MRI and apply the T1w-derived brain mask (SUVR is the quantitative measure). Apply nonlocal mean filtering to field-corrected, intensity-normalized images, then Z-score normalize all volumes and min-max rescale voxel intensities to 0-1, upsampling amyloid PET to 256x256x256 for end-to-end training. The rationale is that the GAN training (batch size 1, image size 256) assumes aligned, intensity-standardized inputs; note the authors caution the normalization step should be avoided when synthesizing quantitative modalities because it distorts quantitative metric values.

## Related tools

- **ANTs (Advanced Normalization Tools)** (N4ITK field-inhomogeneity correction of FLAIR and antsIntermodalityIntrasubject co-registration of FLAIR to T1w)
- **FSL TOPUP** (Eddy-current distortion and involuntary movement correction of diffusion MRI)
- **FSL EDDY** (Eddy-current distortion and involuntary movement correction of diffusion MRI)
- **LONI pipeline (Laboratory of Neuro Imaging pipeline)** (System used to conduct the data processing workflow) — http://pipeline.loni.usc.edu
- **FreeSurfer 5.3.0** (T1w preprocessing and parcellation producing the brain mask applied to PET)
- **SC-GAN** (Downstream consumer of the preprocessed ADNI-3 volumes (256-cube, 0-1 rescaled, 207/58 split)) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python training.py --training_size=207 --img_size=256 --data_dir=data --modalities=flair_t1w_av45 --epoches=120 --lr=0.001
```

## Evaluation signals

- Directory tree matches the README schema: data/train, data/val, data/test, with per-subject modality NIfTI files and target.nii.gz
- Subject split reproduces 265 ADNI-3 subjects divided into 207 training and 58 testing
- All output volumes are 256x256x256 with voxel intensities scaled to 0-1 after Z-score normalization plus min-max rescaling
- PET images are co-registered on T1w MRI with the T1w-derived brain mask applied, and quantitative measurement is based on SUVR
- Downstream training runs without data errors at img_size=256 with batch size 1

## Limitations

- Computational cost is the main limitation of SC-GAN: lengthy training and heavy GPU memory requirements for 256-cube volumes
- The 0-1 min-max normalization negatively affects the quantitative value of DTI metrics, so the normalization step should be avoided when synthesizing quantitative modalities
- The authors' in-house test suggesting no normalization dependency in SC-GAN is not presented and cannot be verified

## Evidence

- [methods] We implemented Z-score normalization for all four tasks, then applied min-max rescaling to scale the voxels’ intensity from between 0 to 1 before the training: "We implemented Z-score normalization for all four tasks, then applied min-max rescaling to scale the voxels’ intensity from between 0 to 1 before the training"
