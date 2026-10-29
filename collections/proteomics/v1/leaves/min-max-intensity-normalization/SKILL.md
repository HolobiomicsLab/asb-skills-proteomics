---
name: min-max-intensity-normalization
description: Use when feeding MRI/PET images into a network trained on min-max-rescaled
  voxel intensities (0–1), or when comparing synthesized PET against ground-truth
  PET so both share the training normalization scale.
when_to_use_negative:
- Synthesizing quantitative modalities (e.g., DTI FA or MD maps) where absolute metric
  values matter — the article states normalization 'negatively affects the quantitative
  value of the metrics' and the step should be avoided
- Images already intensity-normalized (e.g., previously Z-scored or 0–1 rescaled),
  to avoid double scaling
tools:
- name: SC-GAN
  role: Network whose training/testing pipeline requires Z-score normalization followed
    by min-max rescaling of input and target volumes to the 0–1 range
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:909269d185d5dd9ff3c5670727f0e0e5a51dbcfb1452e07a880b4df4873590e0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - FreeSurfer
  - SC-GAN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# min-max-intensity-normalization

## Summary

Rescales voxel intensities (after Z-score normalization) to the 0–1 range so that synthesized and test images (e.g., synthesized amyloid PET from the SC-GAN model) are on the same intensity scale as the training data, enabling comparable quantitative evaluation such as regional SUVR.

## When to use

Use when feeding MRI/PET images into a network trained on min-max-rescaled voxel intensities (0–1), or when comparing synthesized PET against ground-truth PET so both share the training normalization scale.

## When NOT to use

- Synthesizing quantitative modalities (e.g., DTI FA or MD maps) where absolute metric values matter — the article states normalization 'negatively affects the quantitative value of the metrics' and the step should be avoided
- Images already intensity-normalized (e.g., previously Z-scored or 0–1 rescaled), to avoid double scaling

## Inputs

- FLAIR and T1w MRI volumes preprocessed for SC-GAN (N4ITK-corrected, co-registered, nonlocal-mean filtered)
- Ground-truth amyloid PET images (e.g., ADNI-3 av45 PET)
- Synthesized PET images produced by SC-GAN testing

## Outputs

- Intensity-normalized image volumes with voxels scaled to the 0–1 range
- Rescaled synthesized and measured PET volumes ready for SUVR computation per Desikan-Killiany region

## How to apply

First apply Z-score normalization to the input volumes (as done for all four SC-GAN tasks), then apply min-max rescaling to map voxel intensities into the 0–1 interval before training or inference. Apply the identical rescaling to synthesized and ground-truth test images (e.g., the 10 ADNI-3 participants' amyloid PET and their corresponding T1w-derived data) so downstream quantitative measures like UC Berkeley-methodology SUVR per Desikan-Killiany region are computed on a matched intensity scale. When synthesizing quantitative modalities such as DTI-derived FA/MD, consider skipping or reversing the normalization, because the authors note it distorts the quantitative metric values; their in-house test suggested no strong normalization dependency for SC-GAN.

## Related tools

- **SC-GAN** (Network whose training/testing pipeline requires Z-score normalization followed by min-max rescaling of input and target volumes to the 0–1 range) — https://github.com/Haoyulance/SC-GAN

## Examples

```
python testing.py --test_size=10 --gpu=0 --img_size=256 --data_dir=data/test --modalities=flair_t1w_av45 --logdir=logs --output=synthesized_pet  # min-max rescale (0-1) applied to inputs/targets to match training normalization
```

## Evaluation signals

- Normalized volumes have minimum intensity 0 and maximum intensity 1 (check via min/max over the brain-masked volume)
- Synthesized and ground-truth test PET are rescaled with the same normalization as the training PET data before any quantitative comparison
- Downstream SUVR comparison across 110 Desikan-Killiany regions yields significant measured-vs-synthesized correlations (r = 0.67–0.95, P < .0001 in the reference task)
- For DTI tasks, verify whether quantitative FA/MD values remain interpretable; if not, re-run without normalization per the authors' caveat

## Limitations

- The authors note that normalizing target DTI images to 0–1 for stable training negatively affects the quantitative value of DTI metrics, so the normalization step should be avoided when synthesizing quantitative modalities
- The in-house test suggesting no normalization dependency in SC-GAN is not presented in the paper, so it cannot be independently verified
- Higher synthesis error rates occur at high SUVR values, which rescaling does not by itself correct

## Evidence
