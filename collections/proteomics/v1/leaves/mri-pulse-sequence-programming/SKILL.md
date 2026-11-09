---
name: mri-pulse-sequence-programming
description: Use when a CEST (or similar preparation-based) protocol must be defined
  in a shareable .seq file and played out reproducibly on clinical scanners while
  keeping the vendor readout and reconstruction untouched.
when_to_use_negative:
- The readout sequence itself, its UI, or its reconstruction must be modified — the
  SBB design deliberately leaves the established readout and reconstruction untouched
- A universal cross-vendor solution is required today — the interpreter SBB is so
  far only available for Siemens scanners (Philips, United, Canon interpreters are
  still needed)
- The goal is full sequence prototyping including the imaging readout in Pulseq —
  this skill only encapsulates the vendor-defined CEST preparation block inside a
  native readout
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: Pulseq
  role: Open file format and framework used to define RF, gradient, ADC and trigger
    events of the CEST preparation block in a human-readable pulseq-file; interpreter
    source at v1.3.1 is adapted into the SBB
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: Pulseq-CEST
  role: Project providing the CEST preparation .seq library, Bloch-McConnell simulation
    for in-silico validation, and the hybrid interpreter/SBB approach
  repo: https://github.com/kherz/pulseq-cest
- name: Siemens IDEA
  role: Vendor framework for which the hybrid Pulseq-CEST sequence (SBB + readout)
    was implemented and tested on three scanners at three sites
- name: C++
  role: Language in which the Pulseq interpreter is adapted into the SBB module; ~30
    lines of C++ were needed to integrate the SBB into a native EPI sequence
- name: Python (pypulseq-cest)
  role: Python implementation for building CEST saturation blocks with pypulseq and
    simulating them via the C++-based pySimPulseqSBB (BMCTool handling)
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: MATLAB
  role: Used to create pulseq-files and to load/plot and inspect library .seq files
    of CEST preparation periods
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:6d40e7a7568a2e16808387265b6b6fe88880de88b68496d32b887f486589ece4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - MATLAB
  - Pulseq
  - Pulseq-CEST
  - Python (pypulseq-cest)
  - Siemens IDEA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mri-pulse-sequence-programming

## Summary

Program and play out vendor-independent MR sequences by defining pulse sequence events (RF, gradient, ADC, trigger) in the open Pulseq file format and executing them on the scanner via a vendor-specific interpreter, e.g. the Pulseq-CEST sequence building block (SBB) that plays a CEST preparation block ahead of an unchanged readout.

## When to use

Use when a CEST (or similar preparation-based) protocol must be defined in a shareable .seq file and played out reproducibly on clinical scanners while keeping the vendor readout and reconstruction untouched.

## When NOT to use

- The readout sequence itself, its UI, or its reconstruction must be modified — the SBB design deliberately leaves the established readout and reconstruction untouched
- A universal cross-vendor solution is required today — the interpreter SBB is so far only available for Siemens scanners (Philips, United, Canon interpreters are still needed)
- The goal is full sequence prototyping including the imaging readout in Pulseq — this skill only encapsulates the vendor-defined CEST preparation block inside a native readout

## Inputs

- Pulseq v1.3.1 interpreter source code (commit ed8cf4231dc7a5caaf828211486287afcdbac230)
- Pulseq-format .seq file defining the CEST saturation/preparation block (e.g. examples/APTw_3T_example.seq)
- Established vendor readout sequence (e.g. Siemens IDEA 3D snapshot GRE or EPI) into which the SBB is integrated
- Simulation config (yaml) and seq files from the pulseq-cest-library for in-silico Bloch-McConnell testing

## Outputs

- Hybrid Pulseq-CEST sequence (SBB + unchanged readout) executable on a Siemens IDEA scanner
- Measured CEST data (e.g. Z-spectra / APT-weighted images) acquired with a protocol identical to the shared .seq definition
- C++ SBB module that loads RF, gradient and trigger events from the pulseq-file and hands control to the readout at the ADC marker

## How to apply

Define the CEST preparation period (RF saturation pulses, gradients, ADC and trigger events) in a pulseq-file using MATLAB or Python against Pulseq v1.3.1 (commit ed8cf4231dc7a5caaf828211486287afcdbac230), e.g. the example 'APTw_3T_example.seq' from the pulseq-cest repository. Adapt the Pulseq C++ interpreter into a sequence building block (SBB) for the Siemens IDEA framework: remove FOV positioning options (positioning comes from the readout sequence) and load RF, gradient and trigger events from the .seq file. Use the ADC event solely as an internal marker to interrupt the Pulseq playout and switch to the readout — the ADC-containing block is skipped entirely and must contain no other events. Wire the SBB into an established readout (e.g. 3D snapshot GRE or native EPI, where only ~30 lines of C++ code were needed) with minimal initialization, preparation and run calls, leaving timing and SAR calculations to the interpreter layer. Validate that all preparation events execute in order, the ADC-containing block is skipped, and control is handed to the readout without errors.

## Related tools

- **Pulseq** (Open file format and framework used to define RF, gradient, ADC and trigger events of the CEST preparation block in a human-readable pulseq-file; interpreter source at v1.3.1 is adapted into the SBB) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **Pulseq-CEST** (Project providing the CEST preparation .seq library, Bloch-McConnell simulation for in-silico validation, and the hybrid interpreter/SBB approach) — https://github.com/kherz/pulseq-cest
- **Siemens IDEA** (Vendor framework for which the hybrid Pulseq-CEST sequence (SBB + readout) was implemented and tested on three scanners at three sites)
- **C++** (Language in which the Pulseq interpreter is adapted into the SBB module; ~30 lines of C++ were needed to integrate the SBB into a native EPI sequence)
- **Python (pypulseq-cest)** (Python implementation for building CEST saturation blocks with pypulseq and simulating them via the C++-based pySimPulseqSBB (BMCTool handling)) — https://github.com/KerstinHut/pypulseq-cest
- **MATLAB** (Used to create pulseq-files and to load/plot and inspect library .seq files of CEST preparation periods)

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- Unit-test harness plays out the example .seq file against a mock readout and passes only if all preparation events execute in order
- The block containing the ADC event is skipped entirely during playout and the ADC event is used solely as an internal marker to switch to the readout
- The ADC-containing block contains no other events (design invariant of the SBB)
- Integration into an established readout stays minimal (the paper reports ~30 lines of C++ for a native EPI sequence), with readout, UI and reconstruction unchanged
- Simulated Z-spectra from the Bloch-McConnell simulation of the same pulseq-file match measured scanner data (good simulation-measurement match; normalization by the measurement at −6 ppm)

## Limitations

- Source code for the Siemens IDEA interpreter sequence is not publicly available; it is available only upon request
- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for Philips, United and Canon is needed for universal application
- Timing and SAR calculations are left to the interpreter layer, so vendor-side handling of these quantities is required
- Bloch-McConnell based quantification is extremely challenging in vivo

## Evidence

- [intro] a 3D snapshot GRE can be equipped with an encapsulated Pulseq interpreter that solely plays out a CEST preparation block defined in a pulseq-file: "a 3D snapshot GRE26,27 can be equipped with an encapsulated Pulseq interpreter that solely plays out a CEST preparation block defined in a pulseq-file"
- [methods] the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence which is then played out: "the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence which is then played out"
- [methods] to implement the SBB to an established, native EPI sequence only 30 lines of C++ code were necessary, including code regarding communication with the UI: "to implement the SBB to an established, native EPI sequence39,40, only 30 lines of C++ code were necessary, including code regarding communication with the UI"
- [intro] we implemented such a hybrid Pulseq-CEST sequence for the Siemens IDEA framework: "In this work, we implemented such a hybrid Pulseq-CEST sequence for the Siemens IDEA (Integrated Development Environment for Applications, Siemens Healthineers, Erlangen, Germany) framework and"
- [discussion] Source code for the Siemens IDEA interpreter sequence is available upon request.: "Source code for the Siemens IDEA interpreter sequence is available upon request."
- [methods] Download Pulseq as .zip (make sure its 1.3 or commit ed8cf4231dc7a5caaf828211486287afcdbac230): "Download Pulseq as .zip (make sure its 1.3 or commit ed8cf4231dc7a5caaf828211486287afcdbac230)"
