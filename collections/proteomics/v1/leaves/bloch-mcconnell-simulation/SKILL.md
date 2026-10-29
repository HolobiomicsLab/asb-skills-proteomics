---
name: bloch-mcconnell-simulation
description: Use when a CEST protocol is defined as a Pulseq .seq file and you need
  simulated Z-spectra to test or optimize the saturation preparation period in silico,
  or to validate simulations against measured Z-spectra.
when_to_use_negative:
- You need an interpreter implementation for non-Siemens vendors — the Pulseq SBB
  is so far only available for Siemens scanners, though simulation itself remains
  vendor-independent
- You need quantitative in vivo Bloch-McConnell parameter maps — the article notes
  Bloch-McConnell based quantification is extremely challenging in vivo and its CEST-MRF
  is a work-in-progress method
- The saturation block is not defined in a Pulseq .seq file (e.g. proprietary vendor-specific
  sequence definitions)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3300
tools:
- name: Pulseq-CEST
  role: Provides the pulseq-cest-sim Bloch-McConnell simulation used to simulate and
    compare different .seq-files for different settings
  repo: https://github.com/kherz/pulseq-cest
- name: Pulseq
  role: Open file format and reader (ExternalSequence C++ class) defining the CEST
    preparation events that the simulation loops through
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: C++
  role: Native Bloch-McConnell solver (BlochMcConnellSolver.h, SimPulseqSBB.cpp) compiled
    as a MATLAB mex-function or SWIG-wrapped for Python
  repo: https://github.com/kherz/pulseq-cest
- name: MATLAB
  role: Loads .seq files with SequenceSBB and calls the compiled C++ mex-function
    to run simulations
  repo: https://github.com/kherz/pulseq-cest
- name: Python
  role: pySimPulseqSBB (SWIG-wrapped C++) behind the pypulseq_cest parser gives input/output-compatible
    simulations with the MATLAB path
  repo: https://github.com/KerstinHut/pypulseq-cest
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  - task_007
  - task_006
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:a9a843fb29706d5316ea812965c24dac8c91a0f0b407e075513bcf8ef7471d26
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# bloch-mcconnell-simulation

## Summary

Run the pulseq-cest-sim open-source Bloch-McConnell simulation (C++ core exposed as a MATLAB mex-function or SWIG-wrapped Python package) on Pulseq .seq files to compute Z-spectra of CEST preparation periods in silico, so the same file played out on the scanner can be simulated and compared.

## When to use

Use when a CEST protocol is defined as a Pulseq .seq file and you need simulated Z-spectra to test or optimize the saturation preparation period in silico, or to validate simulations against measured Z-spectra.

## When NOT to use

- You need an interpreter implementation for non-Siemens vendors — the Pulseq SBB is so far only available for Siemens scanners, though simulation itself remains vendor-independent
- You need quantitative in vivo Bloch-McConnell parameter maps — the article notes Bloch-McConnell based quantification is extremely challenging in vivo and its CEST-MRF is a work-in-progress method
- The saturation block is not defined in a Pulseq .seq file (e.g. proprietary vendor-specific sequence definitions)

## Inputs

- Pulseq .seq file defining the CEST preparation block (e.g. APTw_3T_001.seq, SLExp_3T_Phantom.seq, MRF_Amide.seq)
- Simulation config YAML with pool settings (CEST pools + semi-solid MTC pool, Lorentzian or Super-Lorentzian line shape) from the pulseq-cest-library sim-library
- Optional measured Z-spectra for comparison (ΔB0-corrected, normalized at −6 ppm)

## Outputs

- Simulated Z-spectrum: one normalized Z value (0–1) per defined saturation offset (e.g. simulated_z_spectrum.csv)
- Final magnetization vector of the water pool after the preparation period (e.g. sim.GetFinalMagnetizationVectors())
- Per-ROI maximum residuals between simulated and measured Z-spectra (e.g. z_spectra_comparison.csv)

## How to apply

Load the .seq file (e.g. examples/APTw_3T_example.seq or SLExp_3T_Phantom.seq) with SequenceSBB in MATLAB or the pypulseq-cest parser and verify the saturation events (e.g. 100 ms HSExp spin-lock pulse, 39 offsets over ±6 ppm). Configure the tissue model as one or more CEST pools plus one semi-solid MTC pool with Lorentzian or Super-Lorentzian line shape, taking pool parameters from the pulseq-cest-library yaml sim-library or Supporting Information tables S2–S4. The C++ application loops through the pulseq-file's sequence events (with 500 samples per pulse) and returns the magnetization vector after the preparation period for each saturation offset. Normalize the Z-spectrum by the value at −6 ppm and compare against measured spectra (ΔB0-corrected by linear interpolation) — e.g. reported maximum residuals of 0.009 (ROI 1), 0.008 (ROI 2), 0.004 (ROI 3). MATLAB and Python implementations must yield identical outputs (Supporting Information Table S5); expect ~0.19 s per single-amide-pool and ~0.77 s per four-pool D-glucose Z-spectrum on an Intel i7-7700K.

## Related tools

- **Pulseq-CEST** (Provides the pulseq-cest-sim Bloch-McConnell simulation used to simulate and compare different .seq-files for different settings) — https://github.com/kherz/pulseq-cest
- **Pulseq** (Open file format and reader (ExternalSequence C++ class) defining the CEST preparation events that the simulation loops through) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **C++** (Native Bloch-McConnell solver (BlochMcConnellSolver.h, SimPulseqSBB.cpp) compiled as a MATLAB mex-function or SWIG-wrapped for Python) — https://github.com/kherz/pulseq-cest
- **MATLAB** (Loads .seq files with SequenceSBB and calls the compiled C++ mex-function to run simulations) — https://github.com/kherz/pulseq-cest
- **Python** (pySimPulseqSBB (SWIG-wrapped C++) behind the pypulseq_cest parser gives input/output-compatible simulations with the MATLAB path) — https://github.com/KerstinHut/pypulseq-cest

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- simulateExample() (or pypulseq_cest.simulate.sim_example()) runs to completion without error
- Output Z-spectrum contains exactly one value per defined saturation offset (e.g. 39 offsets over ±6 ppm) and all normalized Z values fall in 0–1
- Simulated and measured Z-spectra match with maximum residuals on the order of 0.009, 0.008, 0.004 for the phantom ROIs 1–3
- MATLAB and Python simulations of the identical .seq file and pool configuration agree element-wise (Supporting Information Table S5)
- Per-Z-spectrum runtimes are ~0.19 s (single amide pool) and ~0.77 s (four D-glucose pools) at 500 samples per pulse on comparable hardware

## Limitations

- Simulation and measurement agreement is validated qualitatively ('good match') — no numeric acceptance threshold is reported for scanner-comparison use
- The CEST-MRF quantification workflow using these dictionaries is a work-in-progress state undergoing active development
- Runtime benchmarks are hardware-dependent (reported for Intel i7-7700K Kaby Lake); other setups will differ
- Python support requires pre-compiled distributions for python 3.6–3.9 on Linux/Windows; other environments may need manual compilation

## Evidence

- [readme] In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings.: "In this folder you can find the  Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
- [methods] C++ simulation loops through pulseq-file events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] Z-spectrum normalization at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [abstract] Good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [intro] Open-source simulation for scanner playout files: "In addition, we present a fast and flexible open source simulation for the same pulseq-files that are played out on the MR scanner."
