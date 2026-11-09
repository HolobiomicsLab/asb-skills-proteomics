---
name: dti-derived-quantitative-map-normalization
description: Use when quantitative DTI-derived targets (FA or MD maps) or input MR
  images are being prepared for training/testing the SC-GAN or baseline cGANs in the
  ADNI-3 FA/MD/PET synthesis tasks, where stable training requires intensities scaled
  between 0 and 1.
when_to_use_negative:
- When absolute quantitative DTI metric values must be preserved (e.g., clinical MD/FA
  interpretation), since the authors note 'the normalization negatively affects the
  quantitative value of the metrics' and recommend avoiding it for quantitative modalities
- When the input is already intensity-normalized or the task targets non-quantitative
  structural images where rescaling provides no benefit
- When the claimed in-house result of no normalization dependency cannot be verified,
  since 'results are not presented' in the paper
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3306
tools:
- name: SC-GAN
  role: 3D self-attention conditional GAN whose training/testing scripts consume the
    normalized FA/MD and T1w/FLAIR volumes
  repo: https://github.com/Haoyulance/SC-GAN
- name: Quantitative Imaging Toolkit
  role: Fits DTI to diffusion data to produce the FA/MD maps that are subsequently
    normalized
- name: TensorFlow 1.12.2
  role: Framework in which the normalization-preceded GAN training is implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:f4aa65ac05edde7a374477a608ee1f9e22cbdcc02cb2fc964466304dd146a80c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - FreeSurfer
  - Quantitative Imaging Toolkit
  - SC-GAN
  - TensorFlow 1.12.2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# dti-derived-quantitative-map-normalization

## Summary

Applies Z-score normalization followed by min-max rescaling of voxel intensities to the 0-1 range for quantitative DTI-derived maps (fractional anisotropy, mean diffusivity) prior to GAN training in the SC-GAN multimodal synthesis workflow, so that target maps are in a stable range for network optimization.

## When to use

Use when quantitative DTI-derived targets (FA or MD maps) or input MR images are being prepared for training/testing the SC-GAN or baseline cGANs in the ADNI-3 FA/MD/PET synthesis tasks, where stable training requires intensities scaled between 0 and 1.

## When NOT to use

- When absolute quantitative DTI metric values must be preserved (e.g., clinical MD/FA interpretation), since the authors note 'the normalization negatively affects the quantitative value of the metrics' and recommend avoiding it for quantitative modalities
- When the input is already intensity-normalized or the task targets non-quantitative structural images where rescaling provides no benefit
- When the claimed in-house result of no normalization dependency cannot be verified, since 'results are not presented' in the paper

## Inputs

- Preprocessed T1w and FLAIR MRI volumes (.nii.gz, 256x256x256) for ADNI-3 subjects
- Ground-truth quantitative DTI maps (fractional anisotropy, mean diffusivity) fitted with the Quantitative Imaging Toolkit

## Outputs

- Z-score normalized and min-max rescaled (0-1) training/val/test volumes ready for SC-GAN training and testing
- Synthesized FA/MD maps from the testing script requiring inverse normalization for quantitative comparison

## How to apply

For all four synthesis tasks (PET, FA, MD, and superresolution), first apply Z-score normalization to the preprocessed images, then apply min-max rescaling to bring voxel intensities into the 0-1 range before training. Inputs are the preprocessed T1w and FLAIR images and ground-truth DTI-derived maps (e.g., FA from DTI fitting via Quantitative Imaging Toolkit) for the 497 ADNI-3 subjects. Feed the normalized volumes to training.py/testing.py from the SC-GAN repository (default lr=0.001, l1_weight=200, B_weight=200) to synthesize FA/MD maps. The authors' in-house test suggested no normalization dependency in SC-GAN, but normalization was retained in the DTI experiment for stable training. Note the explicit trade-off: rescaling quantitative maps distorts their absolute values, so normalized outputs must be de-normalized (or metrics interpreted carefully) when quantitative DTI values matter.

## Related tools

- **SC-GAN** (3D self-attention conditional GAN whose training/testing scripts consume the normalized FA/MD and T1w/FLAIR volumes) — https://github.com/Haoyulance/SC-GAN
- **Quantitative Imaging Toolkit** (Fits DTI to diffusion data to produce the FA/MD maps that are subsequently normalized)
- **TensorFlow 1.12.2** (Framework in which the normalization-preceded GAN training is implemented)

## Examples

```
python training.py --training_size=398 --gpu=0 --epoches=120 --img_size=256 --data_dir=./data --modalities=flair_t1w_fa --logdir=./logs
```

## Evaluation signals

- Voxel intensities of all training/testing volumes fall within [0, 1] after min-max rescaling
- Synthesized FA volumes achieve whole-brain NRMSE (mean/SD) of 0.078/0.012 with regional NRMSEs of 0.088/0.012 (WM), 0.062/0.01 (GM), 0.073/0.014 (CSF) on 99 ADNI-3 test subjects
- SC-GAN prediction error is 18% lower than 2D conditional GAN for the FA task (acceptance criterion for the metrics ranking)
- Training converges stably with the normalized targets (the stated rationale for including normalization in the DTI experiment)

## Limitations

- Normalizing target DTI images to 0-1 for stable training negatively affects the quantitative value of DTI metrics; the paper recommends avoiding normalization when synthesizing quantitative modalities
- The in-house test suggesting no normalization dependency in SC-GAN is not presented, so it cannot be verified
- Computational cost is the main limitation of SC-GAN: it requires lengthy training and heavy GPU resources, which constrains how often normalization configurations can be re-tuned

## Evidence

- [methods] We implemented Z-score normalization for all four tasks, then applied min-max rescaling to scale the voxels' intensity from between 0 to 1 before the training: [evidence span withheld — anchor status: not_found]
