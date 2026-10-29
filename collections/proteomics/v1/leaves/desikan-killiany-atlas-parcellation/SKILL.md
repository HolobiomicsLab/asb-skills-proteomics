---
name: desikan-killiany-atlas-parcellation
description: Use when regional SUVR quantification of amyloid PET (measured or GAN-synthesized,
  e.g. AV45) is needed over the 110 Desikan-Killiany subcortical and cortical regions,
  with a paired T1w image available for parcellation and masking.
when_to_use_negative:
- No paired T1w MRI is available, so no T1w-derived parcellation or brain mask can
  be computed
- A different cortical atlas or finer subdivision than the 110 Desikan-Killiany regions
  is required
- Absolute quantitative PET intensity (not regional SUVR ratios) is the target, since
  normalized synthesis workflows can distort quantitative voxel values
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: FreeSurfer 5.3.0
  role: Performs T1w preprocessing and parcellation into the Desikan-Killiany regions
    and yields the T1w-derived brain mask applied to co-registered PET
- name: SC-GAN
  role: Generates the synthesized amyloid PET whose regional SUVRs are extracted over
    the parcellated regions and compared to measured PET
  repo: https://github.com/Haoyulance/SC-GAN
- name: ANTs (Advanced Normalization Tools)
  role: N4ITK inhomogeneity correction and antsIntermodalityIntrasubject co-registration
    of modalities to T1w in the surrounding preprocessing chain
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:75131f1bfc67a7fa5696ad7b2864b65cd98680e74baedb751663b1541ddd9a58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - ANTs (Advanced Normalization Tools)
  - FreeSurfer
  - FreeSurfer 5.3.0
  - SC-GAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# desikan-killiany-atlas-parcellation

## Summary

Parcellates T1w MRI into the 110 Desikan-Killiany subcortical and cortical regions with FreeSurfer 5.3.0, and reuses the T1w-derived brain mask and regional labels to extract regional amyloid PET SUVR values for measured vs. synthesized PET comparison.

## When to use

Use when regional SUVR quantification of amyloid PET (measured or GAN-synthesized, e.g. AV45) is needed over the 110 Desikan-Killiany subcortical and cortical regions, with a paired T1w image available for parcellation and masking.

## When NOT to use

- No paired T1w MRI is available, so no T1w-derived parcellation or brain mask can be computed
- A different cortical atlas or finer subdivision than the 110 Desikan-Killiany regions is required
- Absolute quantitative PET intensity (not regional SUVR ratios) is the target, since normalized synthesis workflows can distort quantitative voxel values

## Inputs

- T1w MRI (NIfTI .nii.gz) per subject
- Measured amyloid PET images (e.g. ADNI-3 AV45), co-registered to T1w
- Synthesized PET images (SC-GAN output, .nii.gz), min-max rescaled to training normalization

## Outputs

- FreeSurfer 5.3.0 Desikan-Killiany parcellation of each T1w image (110 subcortical and cortical regions)
- T1w-derived brain mask applied to co-registered PET images
- Per-participant table of 110 regional SUVR values with paired measured-PET and synthesized-PET columns

## How to apply

Run FreeSurfer (ver. 5.3.0) on each subject's T1w image to obtain the Desikan-Killiany parcellation into 110 subcortical and cortical regions and the T1w-derived brain mask. Co-register the PET images (measured amyloid PET and SC-GAN-synthesized PET) onto the T1w MRI, then apply the T1w-derived brain mask to the co-registered PET. Compute the standard uptake value ratio (SUVR) of amyloid-beta uptake per region using the UC Berkeley PET methodology, separately for measured and synthesized PET. Tabulate the 110 regional SUVR pairs per participant so measured-PET and synthesized-PET columns are directly comparable; in the SC-GAN evaluation this was done for 10 randomly selected ADNI-3 participants, with PET intensities min-max rescaled to match training normalization before quantification.

## Related tools

- **FreeSurfer 5.3.0** (Performs T1w preprocessing and parcellation into the Desikan-Killiany regions and yields the T1w-derived brain mask applied to co-registered PET)
- **SC-GAN** (Generates the synthesized amyloid PET whose regional SUVRs are extracted over the parcellated regions and compared to measured PET) — https://github.com/Haoyulance/SC-GAN
- **ANTs (Advanced Normalization Tools)** (N4ITK inhomogeneity correction and antsIntermodalityIntrasubject co-registration of modalities to T1w in the surrounding preprocessing chain)

## Examples

```
recon-all -all -s subject01 -i T1w.nii.gz
```

## Evaluation signals

- Output table contains SUVR values for all 110 Desikan-Killiany regions for each of the 10 test participants
- Each regional row has paired measured-PET and synthesized-PET SUVR columns that are directly comparable
- The brain mask applied to PET originates from the T1w FreeSurfer analysis, not a PET-derived mask
- Both PET volumes were rescaled (min-max) to match the normalization applied to the training PET data before SUVR extraction
- Measured vs. synthesized regional SUVRs show significant correlation (task reported r = 0.67 to r = 0.95, all P < .0001, across all test data)

## Limitations

- Regional SUVR agreement degrades at high SUVR values — the task card reports higher error rates at high SUVRs
- Z-score plus min-max normalization used for SC-GAN training negatively affects the quantitative value of synthesized metrics; the paper notes that for synthesizing quantitative modalities the normalization step should be avoided
- Parcellation quality is tied to the specific FreeSurfer version (5.3.0) used in the study

## Evidence
