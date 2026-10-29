---
name: adc-event-readout-handoff-logic
description: Use when integrating a Pulseq-defined CEST preparation block into an
  existing vendor readout (e.g., 3D snapshot GRE or native EPI) and you need a clean
  mechanism to transfer execution from the interpreter to the native readout.
when_to_use_negative:
- You need the ADC event to acquire data within the Pulseq-defined block — here the
  ADC-containing block is skipped entirely and must contain no other events
- The readout sequence or its reconstruction must be modified — this skill explicitly
  keeps the familiar readout, user interface and reconstruction untouched
- Your target vendor has no Pulseq interpreter available — the SBB is so far only
  available for Siemens scanners
tools:
- name: Pulseq
  role: Source of the v1.3.1 interpreter and open file format whose ADC event is repurposed
    as the handoff marker
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: Pulseq-CEST
  role: Project providing the hybrid sequence concept, example .seq files, and the
    CEST preparation library loaded by the SBB
  repo: https://github.com/kherz/pulseq-cest
- name: C++
  role: Implementation language for the SBB module and the ~30-line integration into
    the native EPI readout under Siemens IDEA
- name: Siemens IDEA
  role: Vendor framework into which the hybrid Pulseq-CEST sequence with the ADC-marker
    handoff was implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:cf89a009aa4a5803ea6dc720823c6d2698bd6154f94c673237117062d24f18f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - Pulseq
  - Pulseq-CEST
  - Siemens IDEA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ADC-event readout handoff logic

## Summary

A design convention for hybrid Pulseq-CEST sequences in which an ADC event inside the pulseq-file is used solely as an internal marker that interrupts the Pulseq preparation playout and hands control to the vendor readout sequence, leaving the established readout and its reconstruction unchanged.

## When to use

Use when integrating a Pulseq-defined CEST preparation block into an existing vendor readout (e.g., 3D snapshot GRE or native EPI) and you need a clean mechanism to transfer execution from the interpreter to the native readout.

## When NOT to use

- You need the ADC event to acquire data within the Pulseq-defined block — here the ADC-containing block is skipped entirely and must contain no other events
- The readout sequence or its reconstruction must be modified — this skill explicitly keeps the familiar readout, user interface and reconstruction untouched
- Your target vendor has no Pulseq interpreter available — the SBB is so far only available for Siemens scanners

## Inputs

- Pulseq .seq file defining the CEST preparation block (e.g., examples/APTw_3T_example.seq)
- Pulseq v1.3.1 interpreter source (commit ed8cf4231dc7a5caaf828211486287afcdbac230)
- Existing vendor readout sequence source (e.g., 3D snapshot GRE or native EPI under Siemens IDEA)

## Outputs

- Compiled C++ sequence building block (SBB) encapsulating the Pulseq interpreter
- Hybrid sequence that plays the CEST preparation block then hands control to the unchanged readout and reconstruction
- Scanner-ready measurement with the original readout pipeline preserved

## How to apply

Adapt the Pulseq v1.3.1 interpreter (commit ed8cf4231dc7a5caaf828211486287afcdbac230) into a C++ sequence building block (SBB) for a vendor framework such as Siemens IDEA, loading RF, gradient and trigger events from the .seq file while removing FOV positioning options (positioning is defined by the readout). When playout reaches an ADC event, treat it exclusively as an internal marker: interrupt the Pulseq sequence and switch to the readout sequence, which is then played out. The block containing the ADC event must be skipped entirely and must not contain any other events, ensuring no preparation hardware is driven during the handoff. Wire the SBB into the readout with minimal initialization, preparation and run calls — the paper reports only ~30 lines of C++ code to integrate the SBB into a native EPI sequence, including UI communication — and leave timing and SAR calculations to the interpreter layer.

## Related tools

- **Pulseq** (Source of the v1.3.1 interpreter and open file format whose ADC event is repurposed as the handoff marker) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **Pulseq-CEST** (Project providing the hybrid sequence concept, example .seq files, and the CEST preparation library loaded by the SBB) — https://github.com/kherz/pulseq-cest
- **C++** (Implementation language for the SBB module and the ~30-line integration into the native EPI readout under Siemens IDEA)
- **Siemens IDEA** (Vendor framework into which the hybrid Pulseq-CEST sequence with the ADC-marker handoff was implemented)

## Evaluation signals

- All preparation events (RF, gradient, trigger) from the .seq file execute in the defined order before the handoff
- The block containing the ADC event is skipped entirely and contains no other events
- Control is handed to the readout sequence without errors and the readout/reconstruction output is identical to the unmodified sequence
- SBB integration code in the readout remains on the order of ~30 lines of C++
- Timing and SAR calculations are handled by the interpreter layer, not duplicated in the readout

## Limitations

- Source code for the Siemens IDEA interpreter sequence is not publicly available; only available upon request
- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for Philips, United or Canon is needed for universal application
- FOV positioning must come from the readout sequence, since positioning options are removed from the interpreter

## Evidence

- [methods] ADC event as handoff marker: "the ADC event is used internally as marker to interrupt the Pulseq sequence and switch to the readout sequence which is then played out"
- [intro] Encapsulated interpreter in 3D snapshot GRE: "a 3D snapshot GRE26,27 can be equipped with an encapsulated Pulseq interpreter that solely plays out a CEST preparation block defined in a pulseq-file"
- [methods] 30-line EPI integration: "to implement the SBB to an established, native EPI sequence39,40, only 30 lines of C++ code were necessary, including code regarding communication with the UI"
- [discussion] Siemens-only availability: "While the SBB is so far only available for Siemens scanners"
