---
name: cpp-sequence-interpreter-implementation
description: Use when a pulseq-file-defined CEST preparation block must be executed
  on a vendor scanner inside an established readout sequence without modifying that
  readout or its reconstruction.
when_to_use_negative:
- The readout sequence itself must be defined or modified in Pulseq format — this
  skill only encapsulates the CEST preparation block; the readout stays vendor-native
- The ADC-containing block in the pulseq-file contains events other than the ADC marker
  — such a file violates the SBB design and cannot be played out correctly
- The target vendor is Philips, United Imaging, or Canon — no interpreter software
  is available for those manufacturers yet
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pulseq
  role: Supplies the v1.3.1 interpreter source and open .seq file format whose RF,
    gradient, ADC, and trigger events the C++ SBB loads and plays out
  repo: https://github.com/pulseq/pulseq
- name: Pulseq-CEST
  role: Provides the pulseq-file library (e.g., APTw_3T_example.seq) used as test
    input for the SBB playout
  repo: https://github.com/kherz/pulseq-cest
- name: Siemens IDEA
  role: Vendor framework in which the hybrid Pulseq-CEST sequence and the C++ SBB
    were implemented and tested on three scanners at three sites
- name: C++
  role: Implementation language for the SBB; integrating the SBB into a native EPI
    sequence required only ~30 lines of C++ code including UI communication
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:b2e3018626aa2315fdc06dc5cfdbbe5eb26d3c1c1e0470f807212c9685029b5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pulseq/pulseq
  tools:
  - C++
  - Pulseq
  - Pulseq-CEST
  - Siemens IDEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cpp-sequence-interpreter-implementation

## Summary

Implement a C++ sequence building block (SBB) that encapsulates a Pulseq interpreter for playing out only a CEST preparation block from a .seq file, embedded in an existing vendor readout sequence (e.g., a Siemens IDEA 3D snapshot GRE or native EPI), leaving the readout, UI, and reconstruction untouched.

## When to use

Use when a pulseq-file-defined CEST preparation block must be executed on a vendor scanner inside an established readout sequence without modifying that readout or its reconstruction.

## When NOT to use

- The readout sequence itself must be defined or modified in Pulseq format — this skill only encapsulates the CEST preparation block; the readout stays vendor-native
- The ADC-containing block in the pulseq-file contains events other than the ADC marker — such a file violates the SBB design and cannot be played out correctly
- The target vendor is Philips, United Imaging, or Canon — no interpreter software is available for those manufacturers yet

## Inputs

- Pulseq v1.3.1 interpreter C++ source (commit ed8cf4231dc7a5caaf828211486287afcdbac230) from pulseq/pulseq
- pulseq-file defining the CEST preparation block (e.g., examples/APTw_3T_example.seq), created with MATLAB or Python
- Established vendor readout sequence source (e.g., Siemens IDEA 3D snapshot GRE or native EPI sequence)

## Outputs

- Compiled C++ Pulseq interpreter SBB integrated into the vendor readout sequence
- Hybrid Pulseq-CEST sequence that plays out the preparation block followed by the unchanged readout on the MR scanner

## How to apply

Obtain the original Pulseq v1.3.1 interpreter source (commit ed8cf4231dc7a5caaf828211486287afcdbac230) and an example .seq file such as 'examples/APTw_3T_example.seq' from the pulseq-cest repository, then adapt the interpreter into a C++ SBB module for the vendor framework (e.g., Siemens IDEA): remove FOV positioning options since positioning is defined by the host readout, and load RF, gradient, and trigger events from the .seq file. Handle ADC events as internal markers only: when the ADC event is reached, interrupt the Pulseq playout, skip the block containing the ADC entirely (it must contain no other events), and hand control to the readout sequence. Wire the SBB into the readout sequence via minimal initialization, preparation, and run calls — for a native EPI sequence this took only about 30 lines of C++ code including UI communication. Leave timing and SAR calculations to the interpreter layer, and validate by playing out the example .seq file against a mock readout, passing only if all preparation events execute in order, the ADC-containing block is skipped, and control is transferred to the readout without errors.

## Related tools

- **Pulseq** (Supplies the v1.3.1 interpreter source and open .seq file format whose RF, gradient, ADC, and trigger events the C++ SBB loads and plays out) — https://github.com/pulseq/pulseq
- **Pulseq-CEST** (Provides the pulseq-file library (e.g., APTw_3T_example.seq) used as test input for the SBB playout) — https://github.com/kherz/pulseq-cest
- **Siemens IDEA** (Vendor framework in which the hybrid Pulseq-CEST sequence and the C++ SBB were implemented and tested on three scanners at three sites)
- **C++** (Implementation language for the SBB; integrating the SBB into a native EPI sequence required only ~30 lines of C++ code including UI communication)

## Evaluation signals

- All preparation events from the .seq file execute in order during mock-readout playout
- The block containing the ADC event is skipped entirely and contains no other executed events
- Control is handed to the readout sequence without errors after the ADC marker interrupts the Pulseq playout
- Readout sequence, user interface, and reconstruction remain byte-identical to the original established sequence
- SBB integration code volume is minimal (the article reports ~30 lines of C++ for a native EPI sequence)

## Limitations

- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for Philips, United, or Canon is needed for universal application
- Source code for the Siemens IDEA interpreter sequence is not publicly available; it is only available upon request
- The ADC event is usable solely as an internal marker — the block containing it must not contain any other events, and FOV positioning options must be removed because positioning is defined by the readout sequence

## Evidence

- [methods] ADC marker interrupts playout: "the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence which is then played out"
- [methods] Minimal integration effort: "to implement the SBB to an established, native EPI sequence39,40, only 30 lines of C++ code were necessary, including code regarding communication with the UI"
- [intro] Encapsulated interpreter in readout: "a 3D snapshot GRE26,27 can be equipped with an encapsulated Pulseq interpreter that solely plays out a CEST preparation block defined in a pulseq-file"
- [discussion] Vendor portability requirement: "A design of the interpreter software for other manufacturers (e.g. Philips, United or Canon) is needed for a universal application."
- [methods] Pulseq version pinning: "Download Pulseq as .zip (make sure its 1.3 or commit ed8cf4231dc7a5caaf828211486287afcdbac230)"
