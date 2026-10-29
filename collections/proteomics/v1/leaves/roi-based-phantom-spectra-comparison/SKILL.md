---
name: roi-based-phantom-spectra-comparison
description: Use when a shared .seq CEST preparation file and phantom (or measured)
  Z-spectra must be validated by simulating Z-spectra per ROI and comparing residuals
  against reported agreement values.
when_to_use_negative:
- The aim is in-vivo Bloch-McConnell based quantification — the article notes this
  is extremely challenging in vivo and out of scope
- The acquisition lacks measured Z-spectra for the phantom ROIs, so no residual comparison
  can be computed
- The protocol requires vendor-specific interpreter behavior on non-Siemens scanners
  — the Pulseq SBB is so far only available for Siemens scanners
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3300
tools:
- name: Pulseq-CEST
  role: Framework providing the SequenceSBB MATLAB object and the Bloch-McConnell
    simulation used to simulate the .seq preparation files
  repo: https://github.com/kherz/pulseq-cest
- name: Pulseq
  role: Open, vendor-independent file format in which the phantom saturation protocol
    (.seq file) is defined
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: MATLAB
  role: Used to read the .seq file into the SequenceSBB object and to call the mex-wrapped
    C++ simulator
- name: C++
  role: Language of the pulseq-cest-sim Bloch-McConnell simulator that loops through
    the pulseq-file sequence events and returns the final magnetization
  repo: https://github.com/kherz/pulseq-cest
- name: Python
  role: Alternative route via pypulseq-cest/pySimPulseqSBB for seq-file parsing and
    simulation of the same .seq files
  repo: https://github.com/KerstinHut/pypulseq-cest
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:1233ad39a90f78675a3df75a845765812c74efa46df70a91334a6cbae497915d
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# roi-based-phantom-spectra-comparison

## Summary

Reproduce the reported agreement between pulseq-cest Bloch-McConnell simulated and measured Z-spectra on an L-arginine/D-glucose phantom, ROI by ROI, quantified by per-ROI maximum residuals (~0.009, 0.008, 0.004).

## When to use

Use when a shared .seq CEST preparation file and phantom (or measured) Z-spectra must be validated by simulating Z-spectra per ROI and comparing residuals against reported agreement values.

## When NOT to use

- The aim is in-vivo Bloch-McConnell based quantification — the article notes this is extremely challenging in vivo and out of scope
- The acquisition lacks measured Z-spectra for the phantom ROIs, so no residual comparison can be computed
- The protocol requires vendor-specific interpreter behavior on non-Siemens scanners — the Pulseq SBB is so far only available for Siemens scanners

## Inputs

- Phantom preparation .seq file (e.g. SLExp_3T_Phantom.seq) in the Pulseq open file format
- Per-ROI pool and tissue parameter tables (SI Tables S2–S4) in yaml format for the Bloch-McConnell simulation
- Measured Z-spectra for the three phantom ROIs (L-arginine and D-glucose tubes)

## Outputs

- Simulated Z-spectra for each of the three phantom ROIs (39 offsets, ±6 ppm)
- Normalized (−6 ppm reference) simulated and measured Z-spectra per ROI
- Per-ROI maximum residuals written to z_spectra_comparison.csv (expected ~0.009, 0.008, 0.004)

## How to apply

Read the phantom preparation definition (e.g. SLExp_3T_Phantom.seq with a 100 ms HSExp saturation pulse and 39 offsets covering ±6 ppm) into the MATLAB SequenceSBB object and inspect it. Load the pool and tissue parameters for each phantom ROI from the supporting information tables (S2–S4) — water pool plus solute CEST pools with Lorentzian/super-Lorentzian MTC line shapes — and run the pulseq-cest-sim C++ Bloch-McConnell simulator (via the MATLAB mex-function) over the 39 saturation offsets for each ROI. Normalize both simulated and measured Z-spectra by the signal at −6 ppm, then compute the maximum residual per ROI. Target agreement is maximum residuals of ~0.009 (ROI 1), ~0.008 (ROI 2), and ~0.004 (ROI 3, the single amide CEST pool and D-glucose tube ROIs), which validates the exact-definition-plus-flexible-simulation approach.

## Related tools

- **Pulseq-CEST** (Framework providing the SequenceSBB MATLAB object and the Bloch-McConnell simulation used to simulate the .seq preparation files) — https://github.com/kherz/pulseq-cest
- **Pulseq** (Open, vendor-independent file format in which the phantom saturation protocol (.seq file) is defined) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **MATLAB** (Used to read the .seq file into the SequenceSBB object and to call the mex-wrapped C++ simulator)
- **C++** (Language of the pulseq-cest-sim Bloch-McConnell simulator that loops through the pulseq-file sequence events and returns the final magnetization) — https://github.com/kherz/pulseq-cest
- **Python** (Alternative route via pypulseq-cest/pySimPulseqSBB for seq-file parsing and simulation of the same .seq files) — https://github.com/KerstinHut/pypulseq-cest

## Examples

```
sim = simulate(config_file='sim-library/SLExp_3T_Phantom_ROI.yaml', seq_file='seq-library/SLExp_3T_Phantom.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- z_spectra_comparison.csv exists and contains Z-spectra columns for all three ROIs and both simulated and measured data
- Per-ROI maximum residuals are on the order of 0.009 (ROI 1), 0.008 (ROI 2), and 0.004 (ROI 3)
- Both simulated and measured Z-spectra are normalized by the measurement at −6 ppm before comparison
- The simulated offset grid matches the 39 saturation offsets of the .seq file covering ±6 ppm
- Simulated spectra use the ROI-specific pool parameters (water pool plus solute CEST pools, Lorentzian/super-Lorentzian MTC line shapes) from SI Tables S2–S4

## Limitations

- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for other manufacturers (Philips, United, Canon) is needed for universal application
- Source code for the Siemens IDEA interpreter sequence is only available upon request, limiting full reproduction of measured spectra on the acquisition side
- Bloch-McConnell based quantification is extremely challenging in vivo; this skill validates simulation-measurement match on phantoms only
- Results depend on the .seq file being readable in the compatible Pulseq version (v1.3 or commit ed8cf4231dc7a5caaf828211486287afcdbac230)

## Evidence

- [results] Maximum residuals of ~0.009, 0.008, 0.004 per ROI (source card finding): [evidence span withheld — anchor status: not_found]
- [methods] Normalization convention at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [methods] C++ Bloch-McConnell simulation of pulseq-file events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] Phantom composition of L-arginine and D-glucose tubes: "A phantom was prepared using multiple 6 ml tubes with either L-arginine or D-glucose"
- [abstract] Good match between simulation and measurement due to exact definition (abstract claim): "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
