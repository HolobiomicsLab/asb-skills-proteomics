---
name: medical-image-intensity-normalization
description: Use when preparing high-resolution multimodal MRI volumes (e.g., T1w/FLAIR
  inputs and synthesis targets like MD, FA, or PET) for deep-learning training that
  requires stable voxel-intensity ranges before optimization.
when_to_use_negative:
- When synthesizing quantitative modalities (e.g., DTI-derived maps) where the quantitative
  value of downstream metrics matters — normalization to 0–1 negatively affects these
  values
- When the downstream analysis requires preserving absolute intensity units (e.g.,
  SUVR-based quantitative PET measurement) rather than a normalized 0–1 range
tools:
- name: SC-GAN
  role: Deep-learning model trained on the Z-score normalized, 0–1 min-max rescaled
    volumes; the normalization was included in the DTI experiment per the paper's
    discussion
  repo: https://github.com/Haoyulance/SC-GAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28819
    title: Three‐dimensional self‐attention conditional GAN with spectral normalization
      for multimodal neuroimaging synthesis
  source_content_hash: sha256:6ed59a83e0eaf2a4847d30fc5c173791451cbd804b2c293177da2b779563e862
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Haoyulance/SC-GAN
  tools:
  - SC-GAN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# medical-image-intensity-normalization

## Summary

Intensity normalization of multimodal neuroimaging volumes (T1w, FLAIR, and target maps such as MD) using Z-score normalization followed by min-max rescaling to the 0–1 range, as applied before training GAN-based synthesis networks on 256 x 256 x 256 ADNI-3 data.

## When to use

Use when preparing high-resolution multimodal MRI volumes (e.g., T1w/FLAIR inputs and synthesis targets like MD, FA, or PET) for deep-learning training that requires stable voxel-intensity ranges before optimization.

## When NOT to use

- When synthesizing quantitative modalities (e.g., DTI-derived maps) where the quantitative value of downstream metrics matters — normalization to 0–1 negatively affects these values
- When the downstream analysis requires preserving absolute intensity units (e.g., SUVR-based quantitative PET measurement) rather than a normalized 0–1 range

## Inputs

- Preprocessed NIfTI volumes (.nii.gz) of input modalities T1w and FLAIR, up-sampled to 256 x 256 x 256
- Target modality volumes (e.g., MD maps) preprocessed via N4ITK correction, nonlocal-mean filtering, DTI fitting, and co-registration

## Outputs

- Z-score normalized and min-max rescaled (0–1) voxel-intensity volumes per subject and modality, ready for GAN training/testing

## How to apply

For each volume in the training set, first apply Z-score normalization (subtract the volume mean and divide by its standard deviation), then apply min-max rescaling to bring voxel intensities into the 0–1 range before training. In the SC-GAN workflow this normalization was implemented for all four tasks (PET, FA, MD synthesis and the MUDI superresolution application) on ADNI-3 volumes up-sampled to 256 x 256 x 256. Apply the normalization per volume (per subject/modality) so that inputs from different scanners and modalities share a comparable range for the generator and discriminator. Note the decision point: for synthesizing quantitative modalities such as DTI-derived maps, the paper warns that min-max rescaling to 0–1 negatively affects the quantitative value of the metrics, so the normalization step should be avoided when quantitative DTI values must be preserved.

## Related tools

- **SC-GAN** (Deep-learning model trained on the Z-score normalized, 0–1 min-max rescaled volumes; the normalization was included in the DTI experiment per the paper's discussion) — https://github.com/Haoyulance/SC-GAN

## Evaluation signals

- After Z-score normalization, each output volume should have approximately zero mean and unit standard deviation
- After min-max rescaling, all voxel intensities should lie within the 0–1 range before training
- The same normalization procedure should be applied consistently across all tasks (all four tasks used Z-score normalization followed by min-max rescaling)
- If synthesizing quantitative DTI metrics, verify that the rescaling step was skipped or that the metrics account for the distortion, since normalization negatively affects quantitative metric values

## Limitations

- Normalizing target DTI images to 0–1 for stable training negatively affects the quantitative value of DTI metrics; the paper recommends avoiding normalization when synthesizing quantitative modalities
- The in-house test suggesting no normalization dependency in SC-GAN is not presented in the paper, so the need for normalization in the DTI experiment cannot be independently verified
- Z-score normalization must be applied per-volume; per-volume statistics depend on the specific acquisition, so cross-site shifts in mean/SD will change the normalized output

## Evidence

- [readme] README data format for training inputs: [evidence span withheld — anchor status: not_found]
