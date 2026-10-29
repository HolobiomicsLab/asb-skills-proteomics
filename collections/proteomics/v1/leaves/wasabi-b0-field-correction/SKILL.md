---
name: wasabi-b0-field-correction
description: Use when CEST Z-spectral imaging volumes (e.g. APTw pulseq-file acquisitions
  at 3 T) require B0 inhomogeneity correction before voxel-wise MTRasym(3.5 ppm) extraction,
  as in healthy-volunteer or tumor-patient in vivo data.
when_to_use_negative:
- When no WASABI (or equivalent B0-mapping) measurement was acquired — the ΔB0 estimate
  and linear interpolation cannot be computed
- When the goal is Bloch-McConnell-based quantification of absolute CEST parameters
  rather than contrast mapping, since this is stated to be extremely challenging in
  vivo
tools:
- name: Pulseq-CEST
  role: MATLAB/Python toolchain used to load the pulseq-files and acquired Z-spectral
    imaging data prior to WASABI-based ΔB0 correction
  repo: https://github.com/kherz/pulseq-cest
- name: Pulseq
  role: Open file format defining the APTw preparation sequences (APTw_3T_001/002/003_AVG.seq)
    whose Z-spectra are corrected
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:95916c99632804ca9e646c34ef5862d6137285c87fec5b7213b01d4382d2e153
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - Pulseq
  - Pulseq-CEST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# WASABI B0 field correction for CEST Z-spectra

## Summary

Corrects CEST Z-spectral imaging data for B0 field inhomogeneity using the WASABI measurement, with ΔB0 corrected by linear interpolation between acquired frequency offsets, so that MTRasym(3.5 ppm) maps are computed on shifted-corrected Z-spectra.

## When to use

Use when CEST Z-spectral imaging volumes (e.g. APTw pulseq-file acquisitions at 3 T) require B0 inhomogeneity correction before voxel-wise MTRasym(3.5 ppm) extraction, as in healthy-volunteer or tumor-patient in vivo data.

## When NOT to use

- When no WASABI (or equivalent B0-mapping) measurement was acquired — the ΔB0 estimate and linear interpolation cannot be computed
- When the goal is Bloch-McConnell-based quantification of absolute CEST parameters rather than contrast mapping, since this is stated to be extremely challenging in vivo

## Inputs

- CEST Z-spectral imaging volumes acquired with pulseq-files (e.g. APTw_3T_001_AVG.seq, APTw_3T_002_AVG.seq, APTw_3T_003_AVG.seq)
- WASABI measurement data for ΔB0 estimation
- Motion-corrected in-vivo volume stack (elastix, Rigid_MMI.txt parameters)

## Outputs

- ΔB0-corrected Z-spectra per voxel, resampled by linear interpolation between acquired offsets
- Corrected images ready for PCA denoising (Malinowski criterion), Gaussian smoothing (σ = 0.6), and S0/−6 ppm normalization
- MTRasym(3.5 ppm) maps for each of the three consensus APTw protocols

## How to apply

Acquire or load the WASABI measurement alongside the Z-spectral imaging data (e.g. the shared APTw_3T_00x_AVG.seq pulseq-files read via the Pulseq-CEST MATLAB/Python toolchain). After motion-correcting all in-vivo volumes with elastix (Rigid_MMI.txt), fit the WASABI data to estimate the voxel-wise ΔB0 map. Correct each Z-spectrum for the estimated field shift, using linear interpolation between acquired frequency offsets to resample intensities at the nominal offsets. The corrected Z-spectra are then denoised (PCA with the Malinowski criterion), smoothed with a 2D in-plane Gaussian filter (σ = 0.6), normalized to S0/−6 ppm, and used for MTRasym(Δω) = (S(−Δω)−S(Δω))/S0 computation. Rationale: CEST contrast around 3.5 ppm is highly sensitive to B0 shifts, so correction must precede asymmetry analysis; both simulated (Bloch-McConnell of the same .seq files) and measured Z-spectra should be processed identically for comparison.

## Related tools

- **Pulseq-CEST** (MATLAB/Python toolchain used to load the pulseq-files and acquired Z-spectral imaging data prior to WASABI-based ΔB0 correction) — https://github.com/kherz/pulseq-cest
- **Pulseq** (Open file format defining the APTw preparation sequences (APTw_3T_001/002/003_AVG.seq) whose Z-spectra are corrected) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1

## Evaluation signals

- Simulated Z-spectra (Bloch-McConnell of the same .seq files) and measured Z-spectra, both normalized by the measurement at −6 ppm, show a good match after correction
- Corrected MTRasym(3.5 ppm) maps of the healthy volunteer show similar intensity across all three consensus protocols, as expected
- Corrected MTRasym(3.5 ppm) maps of the glioblastoma patient show clear contrast in the tumor region (Figures 4D–F)
- Averaged MTRasym(3.5 ppm) within the segmented white matter ROI (from the −4 ppm image via SPM) yields tabulated values consistent per protocol

## Limitations

- ΔB0 correction relies on linear interpolation between acquired offsets, so accuracy degrades if offsets are sparsely sampled
- Bloch-McConnell based quantification is extremely challenging in vivo, limiting quantitative (as opposed to contrast-weighted) interpretation
- The Pulseq sequence building block is so far only available for Siemens scanners, restricting where the underlying acquisitions can be reproduced

## Evidence

- [methods] WASABI interpolation correction: "after a ΔB0 correction using a linear interpolation between acquired offsets"
- [methods] Normalization reference: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [methods] PCA denoising: "applying an additional principal component analysis denoising approach47 using the Malinowski criterion"
- [readme] Readme repo role: "Welcome to the **pulseq-cest** repository, a project to faciliate reproducibility in CEST MRI research using the open [*Pulseq*](https://pulseq.github.io/) standard"
