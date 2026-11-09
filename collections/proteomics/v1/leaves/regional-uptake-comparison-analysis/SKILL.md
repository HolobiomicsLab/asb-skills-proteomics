---
name: regional-uptake-comparison-analysis
description: Use when you have paired synthesized and measured amyloid PET images
  (e.g., from MRI-based synthesis) plus T1w images, and need region-level SUVR agreement
  statistics (e.g., r = 0.67–0.95) over ~110 atlas regions.
when_to_use_negative:
- When synthesizing quantitative modalities where normalization distorts metric values
  — the article notes normalization negatively affects quantitative DTI metric values
  and should be avoided for quantitative modalities
- When only voxel-wise image-quality metrics (nRMSE, PSNR, SSIM) are needed rather
  than region-level uptake agreement
- When PET and T1w cannot be co-registered or no T1w-derived brain mask is available
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3444
tools:
- name: SC-GAN
  role: Generates the synthesized PET images whose regional SUVRs are compared against
    ground-truth PET SUVRs
  repo: https://github.com/Haoyulance/SC-GAN
- name: FreeSurfer 5.3.0
  role: Parcellates T1w images into the 110 Desikan-Killiany subcortical and cortical
    regions and provides the T1w-derived brain mask
- name: ANTs (Advanced Normalization Tools)
  role: Provides N4ITK inhomogeneity correction and co-registration of PET images
    on T1w MRI in the broader pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:3dd115044318b1245af73f67ee929659507d7632c1bb5a956a8ff7e43dddc65e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - ANTs (Advanced Normalization Tools)
  - FreeSurfer
  - FreeSurfer 5.3.0
  - SC-GAN
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# regional-uptake-comparison-analysis

## Summary

Compares regional amyloid PET uptake between synthesized and measured (ground-truth) PET by computing SUVR values per Desikan-Killiany atlas region using the UC Berkeley PET methodology, quantifying agreement across subcortical and cortical regions.

## When to use

Use when you have paired synthesized and measured amyloid PET images (e.g., from MRI-based synthesis) plus T1w images, and need region-level SUVR agreement statistics (e.g., r = 0.67–0.95) over ~110 atlas regions.

## When NOT to use

- When synthesizing quantitative modalities where normalization distorts metric values — the article notes normalization negatively affects quantitative DTI metric values and should be avoided for quantitative modalities
- When only voxel-wise image-quality metrics (nRMSE, PSNR, SSIM) are needed rather than region-level uptake agreement
- When PET and T1w cannot be co-registered or no T1w-derived brain mask is available

## Inputs

- Synthesized amyloid PET images (NIfTI, e.g., .nii.gz from SC-GAN synthesis)
- Ground-truth (measured) amyloid PET images (NIfTI)
- T1w anatomical MRI images for parcellation
- FreeSurfer 5.3.0 parcellation of the Desikan-Killiany atlas (110 subcortical and cortical regions)

## Outputs

- Table of 110 regional SUVR values per participant, with paired measured-PET and synthesized-PET columns
- Correlation statistics (r, P values) between synthesized and ground-truth regional SUVRs (expected r = 0.67 to 0.95, all P < .0001)

## How to apply

Load synthesized PET, corresponding ground-truth amyloid PET, and T1w images (e.g., 10 randomly selected ADNI-3 participants), then rescale both PET sets with min-max normalization to match training-data normalization. Parcellate each T1w image into the 110 Desikan-Killiany subcortical and cortical regions with FreeSurfer 5.3.0, and co-register PET images to T1w MRI applying the T1w-derived brain mask. Compute per-region SUVR of amyloid-beta uptake for both measured and synthesized PET using the UC Berkeley PET methodology, then compare the 110 paired SUVR values per participant (e.g., via correlation). Validation requires an output table containing SUVR values for all 110 regions per participant with directly comparable paired measured-PET and synthesized-PET columns.

## Related tools

- **SC-GAN** (Generates the synthesized PET images whose regional SUVRs are compared against ground-truth PET SUVRs) — https://github.com/Haoyulance/SC-GAN
- **FreeSurfer 5.3.0** (Parcellates T1w images into the 110 Desikan-Killiany subcortical and cortical regions and provides the T1w-derived brain mask)
- **ANTs (Advanced Normalization Tools)** (Provides N4ITK inhomogeneity correction and co-registration of PET images on T1w MRI in the broader pipeline)

## Examples

```
python testing.py --test_size=10 --gpu=0 --img_size=256 --data_dir=./data/test --modalities=flair_t1w_av45 --logdir=./logs --output=./synth_pet; then compute 110-region SUVR with FreeSurfer 5.3.0 parcellations
```

## Evaluation signals

- Output table contains SUVR values for all 110 Desikan-Killiany regions for each test participant, with paired measured-PET and synthesized-PET columns directly comparable
- Regional correlation coefficients between synthesized and ground-truth SUVRs fall in the reported range r = 0.67 to 0.95, all at P < .0001
- Both PET sets are min-max rescaled consistently with the normalization applied to training PET data before SUVR computation
- Higher error rates expected at high SUVRs — check that agreement degrades (rather than fails) in high-uptake regions
- PET images are co-registered to T1w MRI and restricted by the T1w-derived brain mask before regional averaging

## Limitations

- Higher error rates occur at high SUVRs, so agreement is weakest for the most amyloid-positive regions
- Synthesized PET is derived from min-max normalized training data; normalization may affect quantitative values of synthesized intensities
- SC-GAN computational cost is a limitation: lengthy training and heavy GPU memory requirements constrain regeneration of the synthesized inputs

## Evidence

- [results] Synthesized PET SUVR values were significantly correlated with ground-truth PET SUVRs across all test data, with correlation coefficients ranging from r = 0.67 to r = 0.95 (all at P < .0001), with higher error rates at high SUVRs.: [evidence span withheld — anchor status: not_found]
