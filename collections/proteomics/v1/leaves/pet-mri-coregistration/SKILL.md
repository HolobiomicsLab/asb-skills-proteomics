---
name: pet-mri-coregistration
description: Use when amyloid PET images must be aligned to T1w MRI for regional SUVR
  quantification, or when validating synthesized PET against measured (ground-truth)
  PET across atlas regions in ADNI-type data.
when_to_use_negative:
- PET images are already co-registered to T1w space with the brain mask applied and
  only raw voxel-wise synthesis (e.g., SC-GAN training) is needed — skip straight
  to Z-score/min-max normalization and training
- The target modality is quantitative DTI (FA/MD) — the article notes normalization
  negatively affects quantitative metric values, so the min-max rescaling step in
  this PET workflow should be avoided there
- Images have uncorrected nonuniform field inhomogeneity — run N4ITK correction (ANTs)
  first, as done in the article's preprocessing
edam_topics:
- http://edamontology.org/topic_3384
tools:
- name: FreeSurfer 5.3.0
  role: Parcellates the T1w image into the 110 Desikan-Killiany subcortical and cortical
    regions used for regional SUVR computation
- name: SC-GAN
  role: Produces the synthesized PET images whose SUVRs are compared against measured
    PET SUVRs after coregistration
  repo: https://github.com/Haoyulance/SC-GAN
- name: ANTs (N4ITK)
  role: Corrects nonuniform field inhomogeneity in images prior to registration/quantification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:d8722f9db9d51cb6ac7aec580568457c39f0586e61bf5115378bf75eeee714e1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - ANTs (N4ITK)
  - FreeSurfer
  - FreeSurfer 5.3.0
  - SC-GAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pet-mri-coregistration

## Summary

Co-registers amyloid PET images (e.g., ADNI av45) onto T1w MRI and computes regional standard uptake value ratios (SUVR) using the UC Berkeley PET methodology over the 110 Desikan-Killiany subcortical and cortical regions, enabling direct comparison of synthesized versus measured PET uptake.

## When to use

Use when amyloid PET images must be aligned to T1w MRI for regional SUVR quantification, or when validating synthesized PET against measured (ground-truth) PET across atlas regions in ADNI-type data.

## When NOT to use

- PET images are already co-registered to T1w space with the brain mask applied and only raw voxel-wise synthesis (e.g., SC-GAN training) is needed — skip straight to Z-score/min-max normalization and training
- The target modality is quantitative DTI (FA/MD) — the article notes normalization negatively affects quantitative metric values, so the min-max rescaling step in this PET workflow should be avoided there
- Images have uncorrected nonuniform field inhomogeneity — run N4ITK correction (ANTs) first, as done in the article's preprocessing

## Inputs

- Amyloid PET images (ADNI-3 av45, frame series with motion artifacts)
- T1w MRI images of the same participants
- Synthesized PET images (e.g., SC-GAN output) when performing agreement validation
- FreeSurfer 5.3.0 parcellation of the T1w image into 110 Desikan-Killiany regions

## Outputs

- Co-registered PET image in T1w space with T1w-derived brain mask applied
- Table of 110 regional SUVR values per participant, with paired measured-PET and synthesized-PET columns
- Regional correlation statistics between synthesized and ground-truth SUVR (r = 0.67–0.95, P < .0001)

## How to apply

First co-register each PET frame to the first extracted frame and combine frames into a single image to lessen subject motion artifacts; then co-register the PET onto the participant's T1w MRI. Parcellate the T1w image into the 110 Desikan-Killiany subcortical and cortical regions using the FreeSurfer 5.3.0 pipeline, and apply the T1w-derived brain mask to the co-registered PET images before quantification. Compute per-region SUVR with the UC Berkeley PET methodology for both the measured and the synthesized (e.g., SC-GAN) PET, keeping paired columns directly comparable. Before any synthesis or comparison, rescale images with min-max normalization (after Z-score normalization) to match the training normalization of the PET data. Correctness is judged by producing a table of 110 regional SUVRs per participant (10 ADNI-3 test participants in the study) with paired measured/synthesized columns, whose correlations ranged from r = 0.67 to r = 0.95 (all P < .0001).

## Related tools

- **FreeSurfer 5.3.0** (Parcellates the T1w image into the 110 Desikan-Killiany subcortical and cortical regions used for regional SUVR computation)
- **SC-GAN** (Produces the synthesized PET images whose SUVRs are compared against measured PET SUVRs after coregistration) — https://github.com/Haoyulance/SC-GAN
- **ANTs (N4ITK)** (Corrects nonuniform field inhomogeneity in images prior to registration/quantification)

## Evaluation signals

- Output table contains SUVR values for all 110 Desikan-Killiany regions for each of the 10 test participants
- Paired measured-PET and synthesized-PET SUVR columns are directly comparable per region and participant
- Regional correlations between synthesized and ground-truth SUVR fall in the reported range r = 0.67 to r = 0.95 (all P < .0001), with higher error at high SUVRs
- The T1w-derived brain mask is applied to the co-registered PET before SUVR quantification
- Both synthesized and test PET are min-max rescaled to match the normalization applied to training PET data (Z-score then min-max to 0–1)

## Limitations

- Higher error rates occur at high SUVRs in the synthesized-vs-ground-truth agreement
- Normalization (Z-score plus min-max rescaling to 0–1) negatively affects quantitative metric values, so quantitative interpretation requires care
- Computational cost is the main limitation of the underlying SC-GAN synthesis: lengthy training and heavy GPU memory requirements

## Evidence
