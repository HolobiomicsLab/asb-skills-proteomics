---
name: vendor-sequence-framework-integration
description: Use when you need to play out a vendor-independent CEST preparation block
  defined in a Pulseq .seq file on a scanner while preserving an existing native readout
  sequence, its UI, and its reconstruction unchanged.
when_to_use_negative:
- The scanner is not Siemens — the SBB is so far only available for Siemens scanners;
  interpreter software for Philips, United or Canon must be designed separately
- You need to modify or re-implement the imaging readout or its reconstruction — the
  SBB deliberately leaves the readout, user interface and reconstruction untouched
- You only need to simulate the CEST preparation block in silico — use the Bloch-McConnell
  simulation instead of building a scanner interpreter
tools:
- name: Pulseq
  role: Open file format and interpreter source (v1.3.1) whose RF, gradient, ADC and
    trigger events are read and played out by the SBB
  repo: https://github.com/pulseq/pulseq
- name: Pulseq-CEST
  role: Repository providing the CEST preparation pulseq-files (e.g. APTw_3T_example.seq)
    that the hybrid sequence playout consumes
  repo: https://github.com/kherz/pulseq-cest
- name: C++
  role: Language in which the SBB interpreter module and its unit-test harness are
    implemented and compiled
- name: Siemens IDEA
  role: Vendor framework into which the hybrid Pulseq-CEST sequence SBB is integrated
    alongside the established readout
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:8b67f37c045cbae6e13d218c8e7e8ab635b1150067792a6e62c7760c71735f82
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# vendor-sequence-framework-integration

## Summary

Reconstruct a hybrid Pulseq-CEST sequence building block (SBB) that encapsulates a Pulseq interpreter for the CEST preparation block inside a vendor framework (Siemens IDEA), so an established readout (e.g., 3D snapshot GRE or native EPI) and its reconstruction remain untouched while the prep block is playout from a .seq file.

## When to use

Use when you need to play out a vendor-independent CEST preparation block defined in a Pulseq .seq file on a scanner while preserving an existing native readout sequence, its UI, and its reconstruction unchanged.

## When NOT to use

- The scanner is not Siemens — the SBB is so far only available for Siemens scanners; interpreter software for Philips, United or Canon must be designed separately
- You need to modify or re-implement the imaging readout or its reconstruction — the SBB deliberately leaves the readout, user interface and reconstruction untouched
- You only need to simulate the CEST preparation block in silico — use the Bloch-McConnell simulation instead of building a scanner interpreter

## Inputs

- Pulseq v1.3.1 interpreter source code (commit ed8cf4231dc7a5caaf828211486287afcdbac230)
- Pulseq .seq file defining the CEST preparation block (e.g. examples/APTw_3T_example.seq)
- Existing vendor readout sequence (e.g. 3D snapshot GRE or native EPI) in the Siemens IDEA framework

## Outputs

- Compiled C++ sequence building block (SBB) that plays out the CEST preparation block from a pulseq-file
- Hybrid sequence in which the readout, user interface and reconstruction are unchanged
- Hand-off from interpreter playout to the native readout sequence at the ADC marker

## How to apply

Obtain the original Pulseq v1.3.1 interpreter source (commit ed8cf4231dc7a5caaf828211486287afcdbac230) and an example pulseq-file such as 'examples/APTw_3T_example.seq' from the pulseq-cest repository, then adapt the interpreter into a C++ SBB module for a Siemens-IDEA-style framework, removing FOV positioning options since positioning is defined by the readout sequence. Load RF, gradient, ADC and trigger events from the .seq file and use the ADC event solely as an internal marker to interrupt the Pulseq playout and switch to the readout; the block containing the ADC event is skipped entirely and must contain no other events. Wire the SBB into the established readout with minimal initialization, preparation and run calls — the paper reports only 30 lines of C++ code were needed to integrate the SBB into a native EPI sequence, including UI communication. Leave timing and SAR calculations to the interpreter layer, and validate by playing out the example .seq file and confirming all preparation events execute in order before control is handed to the readout.

## Related tools

- **Pulseq** (Open file format and interpreter source (v1.3.1) whose RF, gradient, ADC and trigger events are read and played out by the SBB) — https://github.com/pulseq/pulseq
- **Pulseq-CEST** (Repository providing the CEST preparation pulseq-files (e.g. APTw_3T_example.seq) that the hybrid sequence playout consumes) — https://github.com/kherz/pulseq-cest
- **C++** (Language in which the SBB interpreter module and its unit-test harness are implemented and compiled)
- **Siemens IDEA** (Vendor framework into which the hybrid Pulseq-CEST sequence SBB is integrated alongside the established readout)

## Evaluation signals

- All preparation events from the .seq file execute in order during playout of the example file against a mock readout
- The block containing the ADC event is skipped entirely and contains no other events
- Control is handed to the native readout sequence without errors after the ADC marker interrupts the Pulseq playout
- The readout, user interface and reconstruction are unchanged (integration code is minimal, ~30 lines for an EPI sequence)
- Timing and SAR calculations are handled by the interpreter layer, not duplicated in the SBB

## Limitations

- The SBB is so far only available for Siemens scanners; a design of the interpreter software for other manufacturers (e.g. Philips, United or Canon) is needed for universal application
- Source code for the Siemens IDEA interpreter sequence is not publicly available in the repository; it is available upon request
- The ADC-containing block must not contain any other events, since the ADC event is used solely as an internal marker to interrupt the Pulseq playout
- FOV positioning options must be removed from the interpreter because positioning is defined by the readout sequence

## Evidence

- [intro] Hybrid sequence design: encapsulated interpreter in readout: "a 3D snapshot GRE26,27 can be equipped with an encapsulated Pulseq interpreter that solely plays out a CEST preparation block defined in a pulseq-file"
- [methods] ADC event as internal marker: "the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence which is then played out"
- [methods] Minimal integration effort: "to implement the SBB to an established, native EPI sequence39,40, only 30 lines of C++ code were necessary, including code regarding communication with the UI"
- [discussion] Vendor availability limitation: "While the SBB is so far only available for Siemens scanners"
- [discussion] Interpreter source availability limitation: "Source code for the Siemens IDEA interpreter sequence is available upon request."
- [methods] Required Pulseq version and commit: "Download Pulseq as .zip (make sure its 1.3 or commit ed8cf4231dc7a5caaf828211486287afcdbac230)"
