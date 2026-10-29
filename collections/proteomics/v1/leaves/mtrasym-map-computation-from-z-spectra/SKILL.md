---
name: mtrasym-map-computation-from-z-spectra
description: Use when the input is acquired Z-spectral imaging data (e.g. from APTw_3T_001/002/003_AVG.seq
  pulseq-files) and the goal is a quantitative MTRasym map at a specific offset such
  as 3.5 ppm.
when_to_use_negative:
- The data are not Z-spectra (e.g. no saturation offsets acquired); MTRasym requires
  mirrored offsets around water and an S0 reference such as the −6 ppm measurement
- The goal is quantitative CEST (Bloch-McConnell based quantification), which the
  article notes is 'extremely challenging in vivo' — MTRasym is a semi-quantitative
  contrast, not an exchange-rate estimate
- Sequences are not defined in the Pulseq standard, since the skill's validation (simulated
  vs. measured Z-spectra) depends on the exact .seq-file definition of the CEST preparation
  block
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3398
tools:
- name: Pulseq-CEST
  role: Toolchain providing the pulseq-files and shared seq/sim libraries from which
    the Z-spectra and the CEST preparation periods are defined and loaded
  repo: https://github.com/kherz/pulseq-cest
- name: Pulseq
  role: Open file format defining the APTw saturation preparation blocks played out
    on the scanner, enabling exact comparison of protocols
  repo: https://github.com/pulseq/pulseq
- name: MATLAB
  role: Environment for the Pulseq-CEST toolchain used to load .seq-files and process
    the Z-spectral data
- name: Python
  role: Alternative implementation (pypulseq-cest) with C++-based Bloch-McConnell
    simulation and parser functions for seq/config handling
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: C++
  role: Bloch-McConnell simulation application looping through pulseq-files to produce
    simulated Z-spectra for validation against measured MTRasym maps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:2afbc8717f446ea25c896c36bb759a7978a6ee59b733c9f72a30ceffa38e348e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - MATLAB
  - Pulseq
  - Pulseq-CEST
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3398
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mtrasym-map-computation-from-z-spectra

## Summary

Computes voxel-wise magnetization transfer ratio asymmetry maps, MTRasym(Δω) = (S(−Δω)−S(Δω))/S0, from B0-corrected and denoised Z-spectral imaging data, e.g. extracting MTRasym(3.5 ppm) for APT-weighted contrast acquired with the three consensus Pulseq-CEST APTw protocols.

## When to use

Use when the input is acquired Z-spectral imaging data (e.g. from APTw_3T_001/002/003_AVG.seq pulseq-files) and the goal is a quantitative MTRasym map at a specific offset such as 3.5 ppm.

## When NOT to use

- The data are not Z-spectra (e.g. no saturation offsets acquired); MTRasym requires mirrored offsets around water and an S0 reference such as the −6 ppm measurement
- The goal is quantitative CEST (Bloch-McConnell based quantification), which the article notes is 'extremely challenging in vivo' — MTRasym is a semi-quantitative contrast, not an exchange-rate estimate
- Sequences are not defined in the Pulseq standard, since the skill's validation (simulated vs. measured Z-spectra) depends on the exact .seq-file definition of the CEST preparation block

## Inputs

- Z-spectral imaging volumes acquired with pulseq-file CEST protocols (e.g. APTw_3T_001_AVG.seq, APTw_3T_002_AVG.seq, APTw_3T_003_AVG.seq; 3 repetitions at the offsets of interest with a dummy scan)
- WASABI B0 measurement for ΔB0 field inhomogeneity correction
- Pulseq-CEST MATLAB/Python toolchain protocol definitions (.seq files)

## Outputs

- Voxel-wise MTRasym(3.5 ppm) maps per protocol
- ROI-averaged MTRasym(3.5 ppm) values (e.g. white matter ROI per protocol for healthy volunteer and glioblastoma patient)
- Normalized Z-spectra (both measured and Bloch-McConnell simulated) for validation

## How to apply

First preprocess the Z-spectral volumes: motion-correct in-vivo volumes with elastix (Rigid_MMI.txt parameter file), correct Z-spectra for B0 field inhomogeneity using the WASABI measurement with ΔB0 correction by linear interpolation between acquired offsets, then denoise with principal component analysis using the Malinowski criterion and smooth with a spatial 2D in-plane Gaussian filter (σ = 0.6). Compute the voxel-wise asymmetry MTRasym(Δω) = (S(−Δω)−S(Δω))/S0 and extract the value at 3.5 ppm, normalizing both simulated and measured Z-spectra by the measurement at −6 ppm. For region-of-interest statistics, segment white matter from the −4 ppm image (e.g. with SPM) and average MTRasym(3.5 ppm) within the WM ROI per protocol. The rationale is that the exact pulseq-defined saturation preparation combined with this standardized postprocessing yields comparable APTw contrast across vendors and sites, and validation relies on a good match between Bloch-McConnell simulated and measured Z-spectra.

## Related tools

- **Pulseq-CEST** (Toolchain providing the pulseq-files and shared seq/sim libraries from which the Z-spectra and the CEST preparation periods are defined and loaded) — https://github.com/kherz/pulseq-cest
- **Pulseq** (Open file format defining the APTw saturation preparation blocks played out on the scanner, enabling exact comparison of protocols) — https://github.com/pulseq/pulseq
- **MATLAB** (Environment for the Pulseq-CEST toolchain used to load .seq-files and process the Z-spectral data)
- **Python** (Alternative implementation (pypulseq-cest) with C++-based Bloch-McConnell simulation and parser functions for seq/config handling) — https://github.com/KerstinHut/pypulseq-cest
- **C++** (Bloch-McConnell simulation application looping through pulseq-files to produce simulated Z-spectra for validation against measured MTRasym maps)

## Examples

```
import numpy as np; mtrasym = (zspec_neg - zspec_pos) / zspec[..., offsets == -6.0]; mtrasym_3p5 = mtrasym[..., np.argmin(np.abs(offsets - 3.5))]
```

## Evaluation signals

- Simulated (Bloch-McConnell of the same .seq files) and measured Z-spectra, both normalized by the −6 ppm measurement, show a good match
- MTRasym maps in the healthy volunteer show similar intensity across all three consensus APTw protocols
- A clear contrast appears in the tumor region of the glioblastoma patient MTRasym(3.5 ppm) maps
- Preprocessing invariants applied before the MTRasym formula: WASABI ΔB0 correction by linear interpolation, PCA denoising with the Malinowski criterion, and 2D in-plane Gaussian smoothing with σ = 0.6
- ROI values are tabulated per protocol (WM ROI segmented from the −4 ppm image) rather than reported as uncorrected raw asymmetry

## Limitations

- The Pulseq sequence building block is so far only available for Siemens scanners; interpreter software for other manufacturers (Philips, United, Canon) is needed for universal application
- Source code for the Siemens IDEA interpreter sequence is available only upon request
- Bloch-McConnell based quantification is extremely challenging in vivo, so MTRasym(3.5 ppm) remains a semi-quantitative contrast

## Evidence

- [other] Compute voxel-wise MTRasym(Δω) = (S(−Δω)−S(Δω))/S0 and extract the MTRasym at 3.5 ppm for each of the three protocols, normalized to the S0/−6 ppm measurement.: [evidence span withheld — anchor status: not_found]
- [methods] Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [methods] applying an additional principal component analysis denoising approach47 using the Malinowski criterion: "applying an additional principal component analysis denoising approach47 using the Malinowski criterion"
- [abstract] The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [readme] In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings.: "In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
