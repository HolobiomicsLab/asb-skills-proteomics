---
name: multi-site-quantitative-map-comparison
description: Use when the goal is to verify that a sophisticated quantitative CEST
  protocol (e.g., spin-lock-based CEST-MRF) yields coherent parameter maps at different
  MR sites/vendors, or to benchmark site-to-site consistency of quantitative MRI maps.
when_to_use_negative:
- The goal is a single-site, qualitative CEST protocol comparison — a simple Z-spectrum
  / MTRasym analysis suffices and dictionary matching is unnecessary overhead
- Only one quantitative map (e.g., APTw ratio) is needed; full Bloch-McConnell dictionary
  quantification is extremely challenging in vivo and beyond the scope for simple
  weighted contrasts
- The scanner is not Siemens — the Pulseq sequence building block (SBB) interpreter
  is so far only available for Siemens scanners
tools:
- name: Pulseq
  role: Open file format for exact, human-readable definition of the CEST preparation
    periods shared across sites
  repo: https://pulseq.github.io/
- name: Pulseq-CEST
  role: Framework providing the seq-library of approved CEST preparation blocks and
    the C++ Bloch-McConnell simulation used to build the MRF dictionaries
  repo: https://github.com/kherz/pulseq-cest
- name: pulseq-cest-library
  role: Repository of pre-defined and approved pre-saturation schemes (.seq) and simulation
    settings (yaml) for cross-site sharing
  repo: https://github.com/kherz/pulseq-cest-library
- name: PyPulseq-CEST
  role: Python implementation for building and simulating CEST saturation blocks via
    BMCTool and pySimPulseqSBB
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: elastix
  role: Motion correction of multisite images and registration to the T1 measurement
- name: SPM
  role: Gray/white matter segmentation masks derived from the synthetic T1-weighted
    image
- name: MATLAB
  role: Loading and inspecting .seq files (e.g., with SequenceSBB) and running the
    simulation/matching pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:a16a72ce66f452bce9e7297493fe4a6312bc77d7819d2d84c019442754b60398
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - MATLAB
  - Pulseq
  - Pulseq-CEST
  - PyPulseq-CEST
  - SPM
  - elastix
  - pulseq-cest-library
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# multi-site-quantitative-map-comparison

## Summary

Reproduce and quantitatively compare CEST-MRF-derived parameter maps (fMT, kMT, fAmide, kAmide) acquired across multiple MR sites and scanner models, using shared Pulseq-CEST sequences and a Bloch-McConnell dictionary matching pipeline, to demonstrate cross-site reproducibility.

## When to use

Use when the goal is to verify that a sophisticated quantitative CEST protocol (e.g., spin-lock-based CEST-MRF) yields coherent parameter maps at different MR sites/vendors, or to benchmark site-to-site consistency of quantitative MRI maps.

## When NOT to use

- The goal is a single-site, qualitative CEST protocol comparison — a simple Z-spectrum / MTRasym analysis suffices and dictionary matching is unnecessary overhead
- Only one quantitative map (e.g., APTw ratio) is needed; full Bloch-McConnell dictionary quantification is extremely challenging in vivo and beyond the scope for simple weighted contrasts
- The scanner is not Siemens — the Pulseq sequence building block (SBB) interpreter is so far only available for Siemens scanners

## Inputs

- Pulseq .seq files (MRF_Amide.seq, MRF_MT.seq, T1prep.seq, T2prep.seq, WASABI.seq)
- Multisite CEST-MRF image data (whole brain EPI readout) from healthy volunteers at each site
- T1 saturation-recovery measurements for registration and segmentation
- Simulation configuration yaml files (e.g., from pulseq-cest-library sim-library)

## Outputs

- Quantitative parameter maps: fMT, kMT, fAmide, kAmide per site
- WASABI-derived B0 and B1 inhomogeneity correction maps
- T1 map and gray/white matter segmentation masks
- Cross-site visual and quantitative comparison (coherence of maps across sites)

## How to apply

Define the CEST preparation periods (e.g., MRF_Amide.seq, MRF_MT.seq with T1prep, T2prep, WASABI modules) as Pulseq .seq files and verify with SequenceSBB that measurement and simulation use identical preparation definitions. Generate Bloch-McConnell dictionaries with the native C++ simulation: an amide-weighted dictionary (31-scan spin-lock schedule, B1 0–4 μT at constant 3.5 ppm offset) and a semi-solid MTC dictionary (B1 0.2–4 μT, offsets 6–14 ppm, Super-Lorentzian line shape). Motion-correct multisite images and register to the T1 measurement with elastix, fit a mono-exponential saturation-recovery T1 map, and derive gray/white matter segmentation masks with SPM. Apply WASABI-derived B0 and B1 inhomogeneity corrections, then perform dictionary-based signal matching (Perlman et al.) to compute fMT, kMT, fAmide and kAmide maps per site. Accept the reproduction when the cross-site maps are visually and quantitatively coherent across all sites and all four parameter maps are produced for every site.

## Related tools

- **Pulseq** (Open file format for exact, human-readable definition of the CEST preparation periods shared across sites) — https://pulseq.github.io/
- **Pulseq-CEST** (Framework providing the seq-library of approved CEST preparation blocks and the C++ Bloch-McConnell simulation used to build the MRF dictionaries) — https://github.com/kherz/pulseq-cest
- **pulseq-cest-library** (Repository of pre-defined and approved pre-saturation schemes (.seq) and simulation settings (yaml) for cross-site sharing) — https://github.com/kherz/pulseq-cest-library
- **PyPulseq-CEST** (Python implementation for building and simulating CEST saturation blocks via BMCTool and pySimPulseqSBB) — https://github.com/KerstinHut/pypulseq-cest
- **elastix** (Motion correction of multisite images and registration to the T1 measurement)
- **SPM** (Gray/white matter segmentation masks derived from the synthetic T1-weighted image)
- **MATLAB** (Loading and inspecting .seq files (e.g., with SequenceSBB) and running the simulation/matching pipeline)

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='config.yaml', seq_file='MRF_MT.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- All four parameter maps (fMT, kMT, fAmide, kAmide) are produced for every site (Tuebingen Prisma, Boston Prisma, Erlangen Trio)
- SequenceSBB confirms the same preparation-period definition is used for both measurement and simulation of the .seq files
- Simulated and measured Z-spectra match after normalization by the measurement at −6 ppm
- Cross-site parameter maps are visually and quantitatively coherent (reproducing the multisite consistency reported in Figure 5 of the paper)
- Gray/white matter segmentation masks are consistent with the synthetic T1-weighted image after elastix registration

## Limitations

- Source code for the Siemens IDEA interpreter sequence is only available upon request, and the Pulseq SBB is so far only available for Siemens scanners; interpreters for Philips, United or Canon are needed for universal application
- Bloch-McConnell based quantification is extremely challenging in vivo, and the presented CEST-MRF method represents a work-in-progress state undergoing active development
- Larger, further studies should compare results with previous CEST-MRF studies; such comparison was beyond the scope of the presented work

## Evidence

- [abstract] abstract: "we could show coherent multisite results for a sophisticated CEST method, highlighting the benefits regarding protocol sharing and reproducibility"
- [methods] methods: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] methods-filter: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [readme] readme: "Here, published and approved CEST preparation periods are made available. This allows exact comparison of CEST preparation periods with newly developed or adapted blocks for reproducible CEST"
- [results] task-finding: [evidence span withheld — anchor status: not_found]
