---
name: pet-surv-quantification
description: Use when regional amyloid-PET uptake must be quantified as SUVR over
  cortical/subcortical parcellations and compared between measured PET and MRI-synthesized
  PET.
when_to_use_negative:
- You need voxel-level image-quality metrics (nRMSE, peak SNR, SSIM) rather than regionally
  averaged uptake ratios
- You are synthesizing quantitative DTI metrics (FA, MD), where the article warns
  that min-max normalization negatively affects the quantitative value of the metrics
  and should be avoided
- No co-registered T1w image is available, so no reliable brain mask or FreeSurfer
  parcellation can be derived
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: FreeSurfer 5.3.0
  role: Performs T1w preprocessing and parcellation into the 110 Desikan-Killiany
    subcortical and cortical regions; its T1w-derived brain mask is applied to the
    co-registered PET images.
- name: SC-GAN
  role: Produces the synthesized PET images from T1w and FLAIR MRI whose regional
    SUVRs are quantified and compared against measured-PET SUVRs.
  repo: https://github.com/Haoyulance/SC-GAN
- name: ANTs (N4ITK module)
  role: Corrects nonuniform field inhomogeneity in upstream FLAIR images feeding the
    multimodal MRI input used for PET synthesis.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:36719c6e441e47fe6fc0a5a07c4c9df8ee06613194429937b12f11027f55dbcc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - ANTs (N4ITK module)
  - FreeSurfer
  - FreeSurfer 5.3.0
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pet-surv-quantification

## Summary

Quantifies regional amyloid-PET uptake as standard uptake value ratios (SUVR) per Desikan-Killiany region by co-registering PET to T1w MRI, applying a T1w-derived brain mask, and parcellating with FreeSurfer 5.3.0. Enables direct paired comparison of SUVRs from measured (ground-truth) vs synthesized (e.g., SC-GAN) PET images.

## When to use

Use when regional amyloid-PET uptake must be quantified as SUVR over cortical/subcortical parcellations and compared between measured PET and MRI-synthesized PET.

## When NOT to use

- You need voxel-level image-quality metrics (nRMSE, peak SNR, SSIM) rather than regionally averaged uptake ratios
- You are synthesizing quantitative DTI metrics (FA, MD), where the article warns that min-max normalization negatively affects the quantitative value of the metrics and should be avoided
- No co-registered T1w image is available, so no reliable brain mask or FreeSurfer parcellation can be derived

## Inputs

- Ground-truth amyloid PET images (NIfTI, e.g., ADNI-3 AV45 PET)
- Synthesized PET images generated from multimodal MRI (e.g., SC-GAN output .nii.gz)
- T1w MRI images used for parcellation and brain-mask derivation
- Participant metadata for 10 randomly selected ADNI-3 test participants

## Outputs

- Table of SUVR values for all 110 Desikan-Killiany regions per participant, with paired measured-PET and synthesized-PET columns
- Regional paired-comparison statistics (correlation coefficients between measured and synthesized SUVRs)

## How to apply

Co-register all PET images onto the participant's T1w MRI, then apply the brain mask obtained from T1w analysis to the co-registered PET. Parcellate the T1w image into the 110 Desikan-Killiany subcortical and cortical regions using FreeSurfer 5.3.0. Rescale both measured and synthesized PET intensities with min-max normalization (0 to 1) to match the normalization applied during SC-GAN training. Compute SUVR per region for both measured and synthesized PET using the UC Berkeley PET methodology, and tabulate the 110 paired regional SUVR values per participant for direct comparison. Success is judged by significant paired correlation across regions (reported r = 0.67 to 0.95, all P < .0001), noting larger errors at high SUVRs.

## Related tools

- **FreeSurfer 5.3.0** (Performs T1w preprocessing and parcellation into the 110 Desikan-Killiany subcortical and cortical regions; its T1w-derived brain mask is applied to the co-registered PET images.)
- **SC-GAN** (Produces the synthesized PET images from T1w and FLAIR MRI whose regional SUVRs are quantified and compared against measured-PET SUVRs.) — https://github.com/Haoyulance/SC-GAN
- **ANTs (N4ITK module)** (Corrects nonuniform field inhomogeneity in upstream FLAIR images feeding the multimodal MRI input used for PET synthesis.)

## Examples

```
python testing.py --test_size=10 --gpu=0 --img_size=256 --data_dir=data/test --modalities=flair_t1w_av45 --logdir=logs --output=synthesized_pet/
```

## Evaluation signals

- Output table contains SUVR values for all 110 Desikan-Killiany regions for each of the 10 test participants
- Each participant has directly comparable paired columns: measured-PET SUVR vs synthesized-PET SUVR
- Paired regional SUVRs show significant correlation (expected r = 0.67 to 0.95, all at P < .0001)
- Residual errors increase at high SUVRs, so agreement checks should stratify by uptake level
- Both PET sets are min-max rescaled to the 0-1 range used during SC-GAN training before quantification

## Limitations

- Higher error rates occur at high SUVRs, limiting agreement for strongly amyloid-positive regions
- Min-max rescaling to 0-1, while needed to match training normalization, distorts absolute quantitative values — the article notes normalization should be avoided when synthesizing quantitative modalities
- SC-GAN synthesis itself is computationally expensive, requiring lengthy training and heavy GPU memory

## Evidence
