---
name: motion-correction-and-registration
description: Use when multislice or multi-volume CEST Z-spectral/MRF image stacks
  show inter-volume subject motion and must be aligned to a reference (e.g. the T1
  measurement) before quantitative map fitting or cross-site comparison.
when_to_use_negative:
- Data are already single-volume or inherently motion-robust (e.g. single-shot 3D
  snapshot GRE volumes with no measurable inter-volume motion)
- The goal is sequence definition or Bloch-McConnell simulation of .seq files — that
  belongs to the Pulseq-CEST simulation skill, not image-space registration
- Non-rigid anatomical deformation is required that rigid mutual-information registration
  cannot model, as Rigid_MMI.txt is a rigid-transform parameter file
edam_operation: http://edamontology.org/operation_3800
tools:
- name: elastix
  role: Rigid motion correction of in-vivo CEST volumes (Rigid_MMI.txt parameter file)
    and registration of multisite images to the T1 measurement
- name: SPM
  role: Gray/white matter segmentation after registration; WM segmentation from the
    −4 ppm image for MTRasym(3.5 ppm) ROI averaging
- name: Pulseq-CEST
  role: Toolchain used to load the acquired Z-spectral imaging data and shared protocol
    .seq files before/around the motion-correction step
  repo: https://github.com/kherz/pulseq-cest
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:2bfa9e0e4c7ee6b5c5111835fe24a5bf50909c31866ff80ee47c34f05bd9fe15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - Pulseq-CEST
  - SPM
  - elastix
  edam_operation: http://edamontology.org/operation_3800
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# motion-correction-and-registration

## Summary

Motion-correct and spatially align multislice/multisite CEST imaging volumes (e.g. Z-spectra or CEST-MRF acquisitions) using elastix, register them to a T1-weighted measurement, and prepare aligned images for downstream B0/B1 correction, dictionary matching, and ROI analysis.

## When to use

Use when multislice or multi-volume CEST Z-spectral/MRF image stacks show inter-volume subject motion and must be aligned to a reference (e.g. the T1 measurement) before quantitative map fitting or cross-site comparison.

## When NOT to use

- Data are already single-volume or inherently motion-robust (e.g. single-shot 3D snapshot GRE volumes with no measurable inter-volume motion)
- The goal is sequence definition or Bloch-McConnell simulation of .seq files — that belongs to the Pulseq-CEST simulation skill, not image-space registration
- Non-rigid anatomical deformation is required that rigid mutual-information registration cannot model, as Rigid_MMI.txt is a rigid-transform parameter file

## Inputs

- Multislice/multi-volume CEST Z-spectral imaging data acquired with shared protocol .seq files (e.g. APTw_3T_001_AVG.seq, APTw_3T_002_AVG.seq, APTw_3T_003_AVG.seq)
- Multisite CEST-MRF image volumes (spin-lock-based MRF_Amide.seq / MRF_MT.seq acquisitions)
- T1 measurement images (saturation-recovery series for mono-exponential T1 fitting)
- elastix parameter file Rigid_MMI.txt

## Outputs

- Motion-corrected image volumes aligned to the reference volume
- Images registered to the T1 measurement for anatomical overlay
- Synthetic T1-weighted image and gray/white matter segmentation masks from SPM
- Spatially consistent data ready for WASABI B0/B1 correction, dictionary-based signal matching, and MTRasym(3.5 ppm) extraction

## How to apply

Motion-correct all in-vivo volumes with elastix — for the consensus APTw protocols use the provided Rigid_MMI.txt parameter file (rigid transform, mutual-information metric) so that only rigid head motion is removed without distorting the Z-spectral signal. In the CEST-MRF multisite workflow, motion-correct the multisite images and register them to the T1 measurement with elastix so that quantitative maps are spatially coherent across sites. After registration, derive anatomical context: fit a mono-exponential saturation-recovery T1 map, generate the synthetic T1-weighted image, and derive gray/white matter segmentation masks with SPM (e.g. segmenting white matter from the −4 ppm image for MTRasym ROI statistics). The rationale is that CEST metrics such as MTRasym(3.5 ppm), fMT, kMT, fAmide and kAmide are computed voxel-wise, so any inter-volume misalignment corrupts voxel-wise comparisons; rigid, information-based registration preserves the acquired saturation contrast while restoring spatial correspondence.

## Related tools

- **elastix** (Rigid motion correction of in-vivo CEST volumes (Rigid_MMI.txt parameter file) and registration of multisite images to the T1 measurement)
- **SPM** (Gray/white matter segmentation after registration; WM segmentation from the −4 ppm image for MTRasym(3.5 ppm) ROI averaging)
- **Pulseq-CEST** (Toolchain used to load the acquired Z-spectral imaging data and shared protocol .seq files before/around the motion-correction step) — https://github.com/kherz/pulseq-cest

## Examples

```
elastix -f T1_reference.nii.gz -m cest_volume.nii.gz -p Rigid_MMI.txt -out reg_out
```

## Evaluation signals

- Motion-corrected volumes show no residual inter-volume translation/rotation on visual inspection of edges and ventricles
- Registered CEST maps overlay coherently on the T1 measurement / synthetic T1-weighted image without anatomical mismatch
- Downstream outputs are computed for every site/volume: all four parameter maps (fMT, kMT, fAmide, kAmide) produced per site, and MTRasym(3.5 ppm) maps generated for all three APTw protocols
- Cross-site comparison remains coherent after registration, matching the multisite consistency (Figure 5) reported for Tuebingen (Prisma), Boston (Prisma) and Erlangen (Trio)
- Simulated (Bloch-McConnell of the same .seq files) and measured Z-spectra, both normalized at −6 ppm, show a good match after the full processing chain

## Limitations

- Bloch-McConnell based quantification is extremely challenging in vivo, so registration quality directly bounds the reliability of fitted maps like fMT and kMT
- The presented CEST-MRF method was in a work-in-progress state undergoing active development, so multisite registration results may change as the protocol matures
- The interpreter/SBB is so far only available for Siemens scanners (Prisma/Trio sites), limiting the multisite contexts in which this workflow applies

## Evidence

- [methods] Motion-correct all in-vivo volumes with elastix using the Rigid_MMI.txt parameter file.: [evidence span withheld — anchor status: not_found]
- [methods] Motion-correct the multisite images and register them to the T1 measurement with elastix.: [evidence span withheld — anchor status: not_found]
- [methods] generate the synthetic T1-weighted image, and derive gray/white matter segmentation masks with SPM: [evidence span withheld — anchor status: not_found]
- [methods] Segment white matter from the −4 ppm image using SPM, average MTRasym(3.5 ppm) within the WM ROI: [evidence span withheld — anchor status: not_found]
- [methods] Motion-correct the multislice Z-spectral imaging data as a prerequisite for WASABI B0 correction.: [evidence span withheld — anchor status: not_found]
