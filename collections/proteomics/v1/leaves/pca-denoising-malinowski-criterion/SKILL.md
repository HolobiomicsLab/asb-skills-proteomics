---
name: pca-denoising-malinowski-criterion
description: Use when processing in-vivo CEST Z-spectral imaging data (e.g., MTRasym(3.5
  ppm) APTw maps) that are too noisy for reliable voxel-wise analysis and need noise
  reduction before MTRasym computation.
when_to_use_negative:
- Data are already high-SNR phantom or simulation Z-spectra where PCA component selection
  could remove true low-signal CEST features
- The Z-spectra have not yet been ΔB0-corrected; motion- and field-corrected volumes
  are the intended input
- The analysis goal is raw Z-spectrum quality assessment rather than production of
  denoised maps for MTRasym computation
tools:
- name: Pulseq-CEST
  role: Toolchain within which the PCA denoising (Malinowski criterion) step is applied
    to the acquired Z-spectral data processed from the shared APTw .seq protocols
  repo: https://github.com/kherz/pulseq-cest
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:e297ff5e4138298b35ef38a54a67c02c7f0886f87f8ea727a95b56dfe12750a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - Pulseq-CEST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# PCA denoising with the Malinowski criterion

## Summary

A voxel-wise principal component analysis denoising filter applied to B0-corrected Z-spectral images in the Pulseq-CEST processing pipeline, using the Malinowski criterion to select the number of retained components before spatial smoothing.

## When to use

Use when processing in-vivo CEST Z-spectral imaging data (e.g., MTRasym(3.5 ppm) APTw maps) that are too noisy for reliable voxel-wise analysis and need noise reduction before MTRasym computation.

## When NOT to use

- Data are already high-SNR phantom or simulation Z-spectra where PCA component selection could remove true low-signal CEST features
- The Z-spectra have not yet been ΔB0-corrected; motion- and field-corrected volumes are the intended input
- The analysis goal is raw Z-spectrum quality assessment rather than production of denoised maps for MTRasym computation

## Inputs

- B0-corrected Z-spectral imaging volumes (Z-spectra at multiple frequency offsets per voxel)
- WASABI-derived ΔB0 maps used to correct Z-spectra before denoising

## Outputs

- Denoised Z-spectral images retaining the Malinowski-selected principal components
- Smoothed (2D in-plane Gaussian, sigma = 0.6) Z-spectral images ready for MTRasym(3.5 ppm) extraction

## How to apply

In the Pulseq-CEST processing chain, apply PCA denoising after B0 inhomogeneity correction (e.g., WASABI correction with linear interpolation between acquired offsets) and before spatial smoothing. Reshape the Z-spectral imaging volume so that voxels with identical acquired offsets form the data matrix, decompose it via principal component analysis, and use the Malinowski criterion to determine how many principal components are signal-bearing versus noise; reconstruct the Z-spectra from the retained components only. Follow the denoising with a spatial 2D in-plane Gaussian filter (sigma = 0.6) to smooth the images. The rationale is that CEST contrast differences such as the tumor MTRasym(3.5 ppm) contrast in the glioblastoma patient depend on small asymmetry values, so noise reduction prior to computing MTRasym(Δω) = (S(−Δω)−S(Δω))/S0 improves the contrast-to-noise of the resulting maps.

## Related tools

- **Pulseq-CEST** (Toolchain within which the PCA denoising (Malinowski criterion) step is applied to the acquired Z-spectral data processed from the shared APTw .seq protocols) — https://github.com/kherz/pulseq-cest

## Evaluation signals

- Denoised and measured Z-spectra, normalized by the measurement at −6 ppm, should show a good match with Bloch-McConnell simulations of the same .seq files
- MTRasym(3.5 ppm) maps of the healthy volunteer should retain similar intensity across the three consensus APTw protocols after denoising
- The tumor-region MTRasym(3.5 ppm) contrast in the glioblastoma patient (Figures 4D–F) should remain clearly visible, not attenuated by over-aggressive component removal
- White matter ROI averages segmented from the −4 ppm image should show plausible per-protocol MTRasym(3.5 ppm) values without denoising-induced artifacts

## Limitations

- The Malinowski criterion component count is data-dependent; too few retained components can suppress genuine CEST asymmetry contrast
- Denoising is applied only after motion correction (elastix, Rigid_MMI.txt) and B0 correction (WASABI with linear interpolation), so applying it earlier on uncorrected data is outside the validated workflow
- The workflow is validated for the three consensus APTw protocols (APTw_3T_001/002/003_AVG.seq) at 3T; transfer to other acquisition schemes is not established here

## Evidence

- [methods] PCA denoising step with Malinowski criterion in the processing chain: "applying an additional principal component analysis denoising approach47 using the Malinowski criterion"
- [methods] Spatial smoothing parameters applied after denoising: "a spatial 2D in-plane Gaussian filter (σ = 0.6) to smooth the images"
- [methods] Normalization convention used for the processed Z-spectra: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [abstract] Validation expectation that processed maps match simulations: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
