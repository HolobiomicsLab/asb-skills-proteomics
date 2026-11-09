---
name: cest-z-spectral-processing
description: Use when raw multislice/multisite CEST-MRF or CEST images need to be
  converted into quantitative maps (fMT, kMT, fAmide, kAmide) with B0/B1 correction
  and dictionary-based signal matching.
when_to_use_negative:
- Input is already a quantitative parameter map (e.g., APTw ratio) that needs no dictionary
  matching
- The sequence cannot be reproduced by the Bloch-McConnell simulation — the article
  notes Bloch-McConnell based quantification is extremely challenging in vivo
- Comparing against previous CEST-MRF studies, which the authors explicitly state
  is beyond the scope of the presented work
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: Pulseq-CEST
  role: Provides the Bloch-McConnell simulation (C++/mex) used to generate the dictionaries
    matched against measured Z-spectra
  repo: https://github.com/kherz/pulseq-cest
- name: MATLAB
  role: Runs the dictionary-based signal matching and Z-spectral processing on the
    pulseq-files and image data
- name: Python
  role: Alternative environment for creating pulseq-files and running simulations
    (pypulseq-cest / BMCTool)
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: C++
  role: Native Bloch-McConnell simulation loops through pulseq-files and returns the
    magnetization vector after preparation
  repo: https://github.com/kherz/pulseq-cest
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:8971b52fc31b6d1f75b8c114ae284d08d21b88211372da29589010d3054f472a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - MATLAB
  - Pulseq-CEST
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cest-z-spectral-processing

## Summary

Processes measured CEST Z-spectra (B0/B1 inhomogeneity correction, denoising, normalization) and converts them into quantitative exchange-parameter maps (fMT, kMT, fAmide, kAmide) via Bloch-McConnell dictionary matching, enabling reproducible multisite CEST-MRF comparisons.

## When to use

Use when raw multislice/multisite CEST-MRF or CEST images need to be converted into quantitative maps (fMT, kMT, fAmide, kAmide) with B0/B1 correction and dictionary-based signal matching.

## When NOT to use

- Input is already a quantitative parameter map (e.g., APTw ratio) that needs no dictionary matching
- The sequence cannot be reproduced by the Bloch-McConnell simulation — the article notes Bloch-McConnell based quantification is extremely challenging in vivo
- Comparing against previous CEST-MRF studies, which the authors explicitly state is beyond the scope of the presented work

## Inputs

- Multisite CEST-MRF image series (e.g., spin-lock-based CEST-MRF with whole brain EPI readout)
- Bloch-McConnell dictionaries generated from the .seq pulseq-files (MRF_Amide.seq, MRF_MT.seq) via the native C++ simulation
- WASABI-derived B0 and B1 inhomogeneity maps
- T1-weighted measurement for registration and segmentation

## Outputs

- Quantitative exchange-parameter maps: fMT, kMT, fAmide, kAmide per site
- B0- and B1-corrected, normalized Z-spectra
- Gray/white matter segmentation masks and mono-exponential saturation-recovery T1 maps

## How to apply

First motion-correct the multisite images and register them to the T1 measurement with elastix; fit a mono-exponential saturation-recovery T1 map, generate a synthetic T1-weighted image, and derive gray/white matter segmentation masks with SPM. Apply WASABI-derived B0 and B1 inhomogeneity corrections to the Z-spectra, and optionally apply principal component analysis denoising using the Malinowski criterion and a spatial 2D in-plane Gaussian filter (σ = 0.6) to smooth the images. Normalize both simulated and measured Z-spectra by the measurement at −6 ppm, with ΔB0 correction using linear interpolation between acquired offsets. Then perform dictionary-based signal matching according to Perlman et al. against precomputed Bloch-McConnell dictionaries (e.g., an amide-weighted MRF dictionary from a 31-scan spin-lock schedule with B1 0–4 μT at 3.5 ppm, and a semi-solid MTC dictionary with B1 0.2–4 μT at offsets 6–14 ppm using a Super-Lorentzian line shape) to compute fMT, kMT, fAmide and kAmide maps for each site.

## Related tools

- **Pulseq-CEST** (Provides the Bloch-McConnell simulation (C++/mex) used to generate the dictionaries matched against measured Z-spectra) — https://github.com/kherz/pulseq-cest
- **MATLAB** (Runs the dictionary-based signal matching and Z-spectral processing on the pulseq-files and image data)
- **Python** (Alternative environment for creating pulseq-files and running simulations (pypulseq-cest / BMCTool)) — https://github.com/KerstinHut/pypulseq-cest
- **C++** (Native Bloch-McConnell simulation loops through pulseq-files and returns the magnetization vector after preparation) — https://github.com/kherz/pulseq-cest

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- All four parameter maps (fMT, kMT, fAmide, kAmide) are produced for every site
- Cross-site maps from Tuebingen (Prisma), Boston (Prisma) and Erlangen (Trio) are visually and quantitatively coherent (matching the Figure 5 multisite consistency)
- Simulated and measured Z-spectra both normalized by the measurement at −6 ppm show a good match
- SequenceSBB confirms the same preparation-period definition is used for measurement and simulation
- PCA denoising follows the Malinowski criterion and Gaussian smoothing uses σ = 0.6

## Limitations

- Bloch-McConnell based quantification is extremely challenging in vivo
- The presented CEST-MRF method is a work-in-progress state undergoing active development
- Comparison with previous CEST-MRF studies was not performed and is beyond the scope of the presented work
- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for Philips, United or Canon is needed for universal application

## Evidence

- [methods] PCA denoising and normalization: "applying an additional principal component analysis denoising approach47 using the Malinowski criterion"
- [methods] Gaussian smoothing: "a spatial 2D in-plane Gaussian filter (σ = 0.6) to smooth the images"
- [methods] B0 correction and normalization: "after a ΔB0 correction using a linear interpolation between acquired offsets"
- [methods] Z-spectrum normalization: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [abstract] Multisite reproducibility finding: "we could show coherent multisite results for a sophisticated CEST method, highlighting the benefits regarding protocol sharing and reproducibility"
- [readme] Simulation handling in C++: "performs Bloch-McConnell simulations for the respective sequence events37, and returns the current magnetization vector after preparation"
