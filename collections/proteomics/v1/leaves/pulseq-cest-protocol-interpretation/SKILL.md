---
name: pulseq-cest-protocol-interpretation
description: Use when reproducing or comparing published CEST protocols (e.g. APTw
  MTRasym(3.5 ppm)) across sites/scanners, or when a CEST preparation scheme must
  be defined once as a .seq-file and then simulated and played out identically.
when_to_use_negative:
- The task requires quantitative Bloch-McConnell-based in-vivo concentration/fitting
  estimates — the article notes Bloch-McConnell based quantification is extremely
  challenging in vivo; use for reproducible preparation and contrast, not absolute
  quantification
- The target scanner is not Siemens and no vendor Pulseq interpreter exists — the
  SBB is so far only available for Siemens scanners; interpreter software for Philips,
  United, or Canon is still needed
- The experiment needs comparison with prior CEST-MRF literature results — the presented
  CEST-MRF method is a work-in-progress state and such comparisons were explicitly
  beyond its scope
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pulseq
  role: Open, vendor-independent file format and framework used to define the CEST
    preparation block as a .seq-file
  repo: https://pulseq.github.io/
- name: Pulseq-CEST
  role: Toolchain (MATLAB/C++ and Python) for building, sharing, simulating, and measuring
    the .seq CEST preparation blocks; provides the pulseq-cest-library of approved
    schemes
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python implementation with C++-based pySimPulseqSBB simulation, BMCTool seq/config
    handling, and parser functions
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: yamlmatlab
  role: MATLAB YAML reader used to load sim-library simulation config files into the
    simulation workflow
  repo: https://github.com/ewiger/yamlmatlab
- name: MATLAB
  role: One of the supported environments for creating pulseq-files and running the
    Bloch-McConnell simulation toolchain
- name: Python
  role: Alternative environment for creating pulseq-files and driving pypulseq-cest
    simulations
- name: C++
  role: Language of the application that loops through pulseq-files and performs Bloch-McConnell
    simulations of the sequence events (SimPulseqSBB)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:84b7547bdfe85ab4e4a3f3c8f5cc52b173812d5c414c37f310bea6fe7b72c282
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - MATLAB
  - Pulseq
  - Pulseq-CEST
  - Python
  - pypulseq-cest
  - yamlmatlab
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pulseq-cest-protocol-interpretation

## Summary

Define, share, simulate, and measure CEST saturation preparation periods as Pulseq .seq-files using the Pulseq-CEST toolchain, enabling multi-site multi-vendor reproducibility of protocols such as the three consensus APTw sequences.

## When to use

Use when reproducing or comparing published CEST protocols (e.g. APTw MTRasym(3.5 ppm)) across sites/scanners, or when a CEST preparation scheme must be defined once as a .seq-file and then simulated and played out identically.

## When NOT to use

- The task requires quantitative Bloch-McConnell-based in-vivo concentration/fitting estimates — the article notes Bloch-McConnell based quantification is extremely challenging in vivo; use for reproducible preparation and contrast, not absolute quantification
- The target scanner is not Siemens and no vendor Pulseq interpreter exists — the SBB is so far only available for Siemens scanners; interpreter software for Philips, United, or Canon is still needed
- The experiment needs comparison with prior CEST-MRF literature results — the presented CEST-MRF method is a work-in-progress state and such comparisons were explicitly beyond its scope

## Inputs

- CEST pre-saturation scheme parameters (offsets, saturation powers/durations) specified in MATLAB or Python
- .seq pulseq-file defining the CEST preparation block (e.g. APTw_3T_001_AVG.seq, APTw_3T_002_AVG.seq, APTw_3T_003_AVG.seq)
- YAML simulation configuration file (sim-library) with Bloch-McConnell tissue/solvent parameters
- Acquired Z-spectral imaging data from one or more scanners/sites

## Outputs

- Shareable, human-readable .seq-file encoding the exact CEST preparation period
- Simulated Z-spectra / final magnetization vectors from the Bloch-McConnell simulation
- Measured Z-spectra and MTRasym maps (e.g. MTRasym(3.5 ppm)) from in-vivo acquisitions
- Protocol-comparison tables (e.g. WM ROI MTRasym(3.5 ppm) per consensus APTw protocol)

## How to apply

Create the CEST preparation block as a human-readable pulseq-file (RF, gradient, ADC, trigger events) using the MATLAB or Python API, and place approved schemes in the pulseq-cest-library with YAML simulation configs from the sim-library. Validate the protocol in silico by looping through the pulseq-file's sequence events in the open-source Bloch-McConnell simulation (C++/pySimPulseqSBB) to predict the water magnetization after preparation. On the scanner, play out the .seq-file via a vendor-specific interpreter: a Pulseq sequence building block (SBB) that plays the preparation, then switches to any readout (e.g. 3D snapshot GRE) at the ADC marker event. For analysis, process acquired Z-spectra (B0 correction, denoising, smoothing) and compute MTRasym(Δω) = (S(−Δω)−S(Δω))/S0, normalized to the −6 ppm measurement, comparing simulated vs. measured Z-spectra for agreement. Keep interpreter versions consistent (Pulseq v1.3.1, commit ed8cf4231dc7a5caaf828211486287afcdbac230) so the exact preparation definition is shared byte-for-byte.

## Related tools

- **Pulseq** (Open, vendor-independent file format and framework used to define the CEST preparation block as a .seq-file) — https://pulseq.github.io/
- **Pulseq-CEST** (Toolchain (MATLAB/C++ and Python) for building, sharing, simulating, and measuring the .seq CEST preparation blocks; provides the pulseq-cest-library of approved schemes) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python implementation with C++-based pySimPulseqSBB simulation, BMCTool seq/config handling, and parser functions) — https://github.com/KerstinHut/pypulseq-cest
- **yamlmatlab** (MATLAB YAML reader used to load sim-library simulation config files into the simulation workflow) — https://github.com/ewiger/yamlmatlab
- **MATLAB** (One of the supported environments for creating pulseq-files and running the Bloch-McConnell simulation toolchain)
- **Python** (Alternative environment for creating pulseq-files and driving pypulseq-cest simulations)
- **C++** (Language of the application that loops through pulseq-files and performs Bloch-McConnell simulations of the sequence events (SimPulseqSBB))

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='sim-library/APTw_3T_001.yaml', seq_file='seq-library/APTw_3T_001_AVG.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- Simulated and measured Z-spectra, both normalized by the measurement at −6 ppm, show a good match for the same .seq-files
- MTRasym(3.5 ppm) maps from the three consensus APTw protocols (3 repetitions at offsets of interest plus a dummy scan) show similar intensity in healthy volunteer WM but clear contrast in the glioblastoma tumor region
- White-matter ROI-averaged MTRasym(3.5 ppm) values are tabulated per protocol and consistent with previously published APTw values (original-author-approved schemes)
- The .seq-file round-trips: it can be loaded in MATLAB for plotting and detailed inspection from the pulseq-cest-library
- The scanner's ADC event correctly acts as the marker interrupting the Pulseq sequence and switching to the readout sequence

## Limitations

- Source code for the Siemens IDEA interpreter sequence is not publicly available; it is only available upon request
- The Pulseq sequence building block (SBB) is so far only available for Siemens scanners; interpreter software for other manufacturers (Philips, United, Canon) is needed for universal application
- Results for the CEST-MRF demonstration application represent a work-in-progress state undergoing active development
- Bloch-McConnell-based quantification is extremely challenging in vivo; the framework supports reproducible preparation and contrast rather than absolute quantification

## Evidence

- [abstract] we propose a CEST definition standard using the open Pulseq format for a shareable, simple, and exact definition of CEST protocols: "we propose a CEST definition standard using the open Pulseq format for a shareable, simple, and exact definition of CEST protocols"
- [methods] an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence: "the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence which is then played out"
- [readme] This library is maintained in a separate repository but is included in the pulseq-cest installation.: "This library is maintained in a [separate repository](https://github.com/kherz/pulseq-cest-library) but is included in the pulseq-cest installation."
- [discussion] While the SBB is so far only available for Siemens scanners: "While the SBB is so far only available for Siemens scanners"
- [methods] to implement the SBB to an established, native EPI sequence, only 30 lines of C++ code were necessary: "to implement the SBB to an established, native EPI sequence39,40, only 30 lines of C++ code were necessary, including code regarding communication with the UI"
