---
name: mri-phantom-simulation-parameter-setup
description: Use when simulating Z-spectra for CEST phantom tubes (L-arginine, D-glucose)
  from a shared .seq preparation file and comparing them to measured Z-spectra to
  reproduce reported ROI-level residuals.
when_to_use_negative:
- The goal is in-vivo Bloch-McConnell based quantification — the article notes it
  is 'extremely challenging in vivo' and out of scope here
- The sequence is not defined in the Pulseq open file format or cannot be read by
  the pulseq-cest simulation tooling
- Only protocol comparison across sites/scanners is needed (e.g., the APTw consensus
  protocols) rather than simulation-measurement matching on phantom ROIs
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3452
tools:
- name: Pulseq-CEST
  role: Provides the pulseq-cest-library (.seq files, YAML sim configs) and pulseq-cest-sim
    Bloch-McConnell simulation used to simulate the phantom preparation and compare
    Z-spectra
  repo: https://github.com/kherz/pulseq-cest
- name: MATLAB
  role: Used to read the .seq file into the SequenceSBB object and call the mex-compiled
    C++ simulator for each ROI parameter set
- name: C++
  role: Application that loops through the pulseq-files and performs the Bloch-McConnell
    simulations for the respective sequence events (mex-function backend)
- name: Pulseq
  role: Open, vendor-independent file format in which the CEST preparation period
    (saturation block) is defined and shared
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: Python
  role: 'Alternative route via pypulseq-cest / BMCTool: simulate(config_file, seq_file,
    normalize=True) returns a SimulationParameters object for retrieving offsets and
    z-magnetization'
  repo: https://github.com/KerstinHut/pypulseq-cest
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:b99b2decb7ad4946025b472f044eb3464e87b115edef2cdb45d2c257d7e540b1
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
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# mri-phantom-simulation-parameter-setup

## Summary

Configure and run Bloch-McConnell simulations of a pulseq-defined CEST preparation (e.g., SLExp_3T_Phantom.seq with 100 ms HSExp saturation over 39 offsets ±6 ppm) against measured Z-spectra from L-arginine and D-glucose phantom tubes, validating agreement via per-ROI maximum residuals (~0.009, ~0.008, ~0.004).

## When to use

Use when simulating Z-spectra for CEST phantom tubes (L-arginine, D-glucose) from a shared .seq preparation file and comparing them to measured Z-spectra to reproduce reported ROI-level residuals.

## When NOT to use

- The goal is in-vivo Bloch-McConnell based quantification — the article notes it is 'extremely challenging in vivo' and out of scope here
- The sequence is not defined in the Pulseq open file format or cannot be read by the pulseq-cest simulation tooling
- Only protocol comparison across sites/scanners is needed (e.g., the APTw consensus protocols) rather than simulation-measurement matching on phantom ROIs

## Inputs

- Pulseq .seq file with the phantom CEST preparation (e.g., SLExp_3T_Phantom.seq: 100 ms HSExp saturation pulse, 39 offsets ±6 ppm)
- Pool and tissue parameter settings for the three phantom ROIs (supporting information tables S2–S4, YAML config)
- Measured Z-spectra of the L-arginine and D-glucose phantom tubes for the three ROIs

## Outputs

- Simulated normalized Z-spectra per ROI (39 offsets)
- Measured normalized Z-spectra per ROI
- Per-ROI maximum residuals (expected ≈0.009, ≈0.008, ≈0.004)
- z_spectra_comparison.csv containing Z-spectra columns for all three ROIs, both simulated and measured data

## How to apply

Read the phantom preparation definition (e.g., SLExp_3T_Phantom.seq with a 100 ms HSExp saturation pulse and 39 offsets covering ±6 ppm) into the MATLAB SequenceSBB object. Load the pool and tissue parameters for the three phantom ROIs from the supporting information tables (S2–S4), covering the water pool plus solute CEST pools with Lorentzian/super-Lorentzian MTC line shapes as required. Execute the pulseq-cest-sim C++ Bloch-McConnell simulator (via the MATLAB mex-function) on the .seq file for each ROI parameter set, sweeping the 39 saturation offsets. Normalize both simulated and measured Z-spectra by the signal at −6 ppm, then compare per ROI and compute the maximum residual (targets ~0.009 for ROI 1, ~0.008 for ROI 2, ~0.004 for ROI 3). Write simulated Z-spectra, measured Z-spectra, and per-ROI maximum residuals to z_spectra_comparison.csv. The rationale is that the exact definition of the CEST preparation period in the pulseq file, combined with the flexible Bloch-McConnell simulation, is what leads to the good match between simulations and measurements.

## Related tools

- **Pulseq-CEST** (Provides the pulseq-cest-library (.seq files, YAML sim configs) and pulseq-cest-sim Bloch-McConnell simulation used to simulate the phantom preparation and compare Z-spectra) — https://github.com/kherz/pulseq-cest
- **MATLAB** (Used to read the .seq file into the SequenceSBB object and call the mex-compiled C++ simulator for each ROI parameter set)
- **C++** (Application that loops through the pulseq-files and performs the Bloch-McConnell simulations for the respective sequence events (mex-function backend))
- **Pulseq** (Open, vendor-independent file format in which the CEST preparation period (saturation block) is defined and shared) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **Python** (Alternative route via pypulseq-cest / BMCTool: simulate(config_file, seq_file, normalize=True) returns a SimulationParameters object for retrieving offsets and z-magnetization) — https://github.com/KerstinHut/pypulseq-cest

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='sim-library/config_Phantom_ROI1.yaml', seq_file='seq-library/SLExp_3T_Phantom.seq', normalize=True)
```

## Evaluation signals

- z_spectra_comparison.csv exists and contains Z-spectra columns for all three ROIs and both simulated and measured data
- Per-ROI maximum residuals are on the order of 0.009 (ROI 1), 0.008 (ROI 2) and 0.004 (ROI 3), confirming a good match between simulation and measurement
- Both simulated and measured Z-spectra are normalized by the measurement at −6 ppm before comparison
- Simulation covers all 39 saturation offsets spanning ±6 ppm from the .seq preparation file
- Simulated offsets and z-magnetization retrieved via get_zspec / GetFinalMagnetizationVectors correspond to the offsets in the .seq file

## Limitations

- The pulseq-cest SBB/interpreter is so far only available for Siemens scanners; interpreter software for other manufacturers (Philips, United, Canon) is needed for universal application
- Source code for the Siemens IDEA interpreter sequence is not publicly available; only available upon request
- The simulation parameters must exactly match the shared preparation definition (pulse shape, duration, offsets) — mismatched pool/tissue settings (tables S2–S4) will change the residuals away from the reported 0.009/0.008/0.004 targets

## Evidence

- [abstract] Exact definition of the CEST preparation period combined with the flexible simulation leads to a good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [methods] Bloch-McConnell simulation of pulseq files in C++: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] Z-spectra normalization by the −6 ppm measurement: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [methods] Phantom with L-arginine and D-glucose tubes: "A phantom was prepared using multiple 6 ml tubes with either L-arginine or D-glucose"
- [readme] pulseq-cest-sim provides the Bloch-McConnell simulation for comparing .seq-files: "In this folder you can find the  Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
