---
name: white-matter-roi-segmentation-spm
description: Use when you need a tissue-defined region of interest (white matter)
  to tabulate or compare MTRasym(3.5 ppm) contrast values from processed CEST/Z-spectral
  data across protocols or subjects.
when_to_use_negative:
- When the goal is quantifying absolute APTw concentration in tumor tissue via Bloch-McConnell
  fitting — the article notes 'Bloch-McConnell based quantification is extremely challenging
  in vivo', so this ROI-averaging approach gives contrast values, not concentration
  estimates
- When no Z-spectral image at the −4 ppm offset (or equivalent tissue-contrast image
  suitable for SPM segmentation) is available
- When the analysis target is a non-brain or non-human dataset where SPM's tissue
  priors for white matter segmentation are inappropriate
tools:
- name: SPM
  role: Segments white matter from the −4 ppm image to define the WM ROI in which
    MTRasym(3.5 ppm) is averaged
- name: MATLAB
  role: Hosts the SPM segmentation and downstream ROI averaging of the MTRasym(3.5
    ppm) maps
- name: Pulseq-CEST
  role: Produces the processed Z-spectral/MTRasym data (via the .seq files APTw_3T_001/002/003_AVG.seq)
    from which the −4 ppm image and MTRasym(3.5 ppm) maps are derived
  repo: github.com/kherz/pulseq-cest
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:b18992c02359fd37cb0501d61235834a124fe53c3614c582c4fe72b6779f5925
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - MATLAB
  - Pulseq-CEST
  - SPM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# White matter ROI segmentation using SPM

## Summary

Segment white matter from a CEST image (the −4 ppm image of the Z-spectrum) with SPM and average the voxel-wise MTRasym(3.5 ppm) values within the resulting WM ROI, enabling quantitative comparison of APTw contrast across consensus protocols in healthy volunteers and glioblastoma patients.

## When to use

Use when you need a tissue-defined region of interest (white matter) to tabulate or compare MTRasym(3.5 ppm) contrast values from processed CEST/Z-spectral data across protocols or subjects.

## When NOT to use

- When the goal is quantifying absolute APTw concentration in tumor tissue via Bloch-McConnell fitting — the article notes 'Bloch-McConnell based quantification is extremely challenging in vivo', so this ROI-averaging approach gives contrast values, not concentration estimates
- When no Z-spectral image at the −4 ppm offset (or equivalent tissue-contrast image suitable for SPM segmentation) is available
- When the analysis target is a non-brain or non-human dataset where SPM's tissue priors for white matter segmentation are inappropriate

## Inputs

- −4 ppm image from the B0-corrected, PCA-denoised, and smoothed Z-spectral dataset (input to SPM segmentation)
- Voxel-wise MTRasym(3.5 ppm) maps normalized to the S0/−6 ppm measurement for each of the three APTw protocols
- SPM segmentation parameters for white matter tissue class

## Outputs

- White matter ROI mask derived from the −4 ppm image
- Per-protocol mean MTRasym(3.5 ppm) values within the WM ROI, tabulated for the healthy volunteer and the glioblastoma patient

## How to apply

After the Z-spectral processing chain (elastix rigid motion correction, WASABI ΔB0 correction with linear interpolation between acquired offsets, PCA denoising via the Malinowski criterion, and 2D in-plane Gaussian smoothing with σ = 0.6), take the −4 ppm image as the input for tissue segmentation. Use SPM to segment white matter from this image, producing a binary or probabilistic WM mask. Apply this mask to the voxel-wise MTRasym maps computed as MTRasym(Δω) = (S(−Δω)−S(Δω))/S0, extracted at 3.5 ppm and normalized to the S0/−6 ppm measurement, and average the masked values per protocol. The rationale is that a tissue-matched ROI (WM) yields comparable contrast numbers between the three consensus APTw protocols (APTw_3T_001_AVG.seq, APTw_3T_002_AVG.seq, APTw_3T_003_AVG.seq) and between the healthy volunteer and the glioblastoma patient, where the tumor region shows clear contrast differences (Figures 4D–F).

## Related tools

- **SPM** (Segments white matter from the −4 ppm image to define the WM ROI in which MTRasym(3.5 ppm) is averaged)
- **MATLAB** (Hosts the SPM segmentation and downstream ROI averaging of the MTRasym(3.5 ppm) maps)
- **Pulseq-CEST** (Produces the processed Z-spectral/MTRasym data (via the .seq files APTw_3T_001/002/003_AVG.seq) from which the −4 ppm image and MTRasym(3.5 ppm) maps are derived) — github.com/kherz/pulseq-cest

## Evaluation signals

- The simulated (Bloch-McConnell of the same .seq files) and measured Z-spectra, both normalized by the measurement at −6 ppm, show a good match, confirming upstream processing before ROI averaging
- Averaged WM ROI MTRasym(3.5 ppm) values can be tabulated per protocol for both the healthy volunteer and the glioblastoma patient
- In the healthy volunteer, MTRasym(3.5 ppm) intensity within the WM ROI is similar for all three consensus protocols, as reported in the source finding
- In the glioblastoma patient, a clear contrast is visible in the tumor region (Figures 4D–F), consistent with the published MTRasym maps
- The WM mask derived from the −4 ppm image plausibly covers white matter anatomy; MTRasym values inside vs. outside the mask should differ consistently with tissue contrast

## Limitations

- The Pulseq SBB is so far only available for Siemens scanners, limiting where the underlying APTw protocols can be acquired
- Source code for the Siemens IDEA interpreter sequence is only available upon request, not publicly released
- Bloch-McConnell based quantification is extremely challenging in vivo, so ROI-averaged MTRasym is a semi-quantitative contrast measure rather than an absolute concentration

## Evidence

- [methods] Segment white matter from the −4 ppm image using SPM, average MTRasym(3.5 ppm) within the WM ROI, and tabulate contrast values per protocol: [evidence span withheld — anchor status: not_found]
- [methods] Compute voxel-wise MTRasym(Δω) = (S(−Δω)−S(Δω))/S0 and extract the MTRasym at 3.5 ppm for each of the three protocols, normalized to the S0/−6 ppm measurement: [evidence span withheld — anchor status: not_found]
- [results] While the intensity in the MTRasym maps of the healthy volunteer is similar for all three protocols, a clear contrast can be seen in the tumor region (Figures 4D–F) of the glioblastoma patient: "While the intensity in the MTRasym maps of the healthy volunteer is similar for all three protocols, a clear contrast can be seen in the tumor region (Figures 4D–F) of the glioblastoma patient."
- [discussion] since Bloch-McConnell based quantification is extremely challenging in vivo: "since Bloch-McConnell based quantification is extremely challenging in vivo"
