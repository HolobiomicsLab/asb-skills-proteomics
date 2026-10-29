---
name: pulseq-sequence-file-parsing
description: Use when the input is a Pulseq .seq text file defining a CEST saturation
  block and you need to read, inspect, plot or feed its events into a Bloch-McConnell
  simulation or scanner interpreter.
when_to_use_negative:
- The input is not a Pulseq-format file but a vendor-native sequence export (e.g.
  Siemens IDEA or Bruker ParaVision raw sequence code) — convert or obtain the .seq
  definition first
- The goal is quantifying CEST contrast from measured images rather than reading a
  protocol definition — use the Bloch-McConnell simulation or Z-spectrum analysis
  instead
- 'Only scanner-side readout timing/SAR details are needed: the Pulseq preparation
  SBB leaves timing and SAR calculations to the interpreter layer, not to the parsed
  preparation file alone'
edam_operation: http://edamontology.org/operation_0336
tools:
- name: Pulseq
  role: Defines the open .seq file format and provides the C++ ExternalSequence class
    used to read the parsed events
  repo: https://github.com/pulseq/pulseq
- name: Pulseq-CEST
  role: Provides the MATLAB SequenceSBB reader for loading and plotting .seq saturation
    blocks; same files feed the Bloch-McConnell simulation
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python parser package (with BMCTool for seq-file/config handling and pySimPulseqSBB
    for the SWIG-wrapped C++ core) giving input/output-compatible parsing and simulation
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: pulseq-cest-library
  role: Source of pre-defined, approved .seq saturation schemes and yaml simulation
    settings that serve as parsing inputs
  repo: https://github.com/kherz/pulseq-cest-library
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_003
  - task_008
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:dc194283a7f1507bede3f507ecc58076d87acb1b840699d0cc5ecbde52234a50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pulseq/pulseq
  tools:
  - C++
  - Pulseq
  - Pulseq-CEST
  - pulseq-cest-library
  - pypulseq-cest
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pulseq-sequence-file-parsing

## Summary

Parses human-readable Pulseq .seq files that define CEST preparation periods (RF, gradient, ADC and trigger events) so they can be inspected, plotted, simulated with Bloch-McConnell, or played out on a scanner via an interpreter sequence.

## When to use

Use when the input is a Pulseq .seq text file defining a CEST saturation block and you need to read, inspect, plot or feed its events into a Bloch-McConnell simulation or scanner interpreter.

## When NOT to use

- The input is not a Pulseq-format file but a vendor-native sequence export (e.g. Siemens IDEA or Bruker ParaVision raw sequence code) — convert or obtain the .seq definition first
- The goal is quantifying CEST contrast from measured images rather than reading a protocol definition — use the Bloch-McConnell simulation or Z-spectrum analysis instead
- Only scanner-side readout timing/SAR details are needed: the Pulseq preparation SBB leaves timing and SAR calculations to the interpreter layer, not to the parsed preparation file alone

## Inputs

- Pulseq .seq text file (e.g. examples/APTw_3T_example.seq, APTw_3T_001/002/003.seq, SLExp_3T_Phantom.seq)
- YAML simulation configuration file from the pulseq-cest-library sim-library (paired with the .seq file in the Python workflow)

## Outputs

- Parsed sequence object (MATLAB SequenceSBB or Python parsed sequence) exposing RF, gradient, ADC and trigger events
- Saturation-event plots (RF magnitude, phase and offset scheduling via plotSaturationPhase)
- Ordered event stream consumed by the C++ Bloch-McConnell solver or the vendor interpreter SBB

## How to apply

Load the .seq file with the pulseq-cest MATLAB SequenceSBB class (e.g. SequenceSBB.read('examples/APTw_3T_example.seq')) and inspect the scheduled saturation events with seq.plotSaturationPhase() to confirm RF magnitude, phase and offset scheduling. The file itself is a text-file in the open Pulseq format in which all sequence parameters — RF pulse, gradient, ADC and trigger events — are defined and which can be created from MATLAB or Python. For the Python path, parse the same file with the pypulseq_cest parser package (BMCTool handles pulseq-cest-specific seq-file and config handling), backed by the C++ ExternalSequence reader that the simulation core uses to loop through the sequence events. The parsed events feed directly into the Bloch-McConnell simulation (which 'performs Bloch-McConnell simulations for the respective sequence events, and returns the current magnetization vector after preparation') or into the vendor interpreter, where the ADC event is used internally as a marker to interrupt the Pulseq playout and switch to the readout sequence.

## Related tools

- **Pulseq** (Defines the open .seq file format and provides the C++ ExternalSequence class used to read the parsed events) — https://github.com/pulseq/pulseq
- **Pulseq-CEST** (Provides the MATLAB SequenceSBB reader for loading and plotting .seq saturation blocks; same files feed the Bloch-McConnell simulation) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python parser package (with BMCTool for seq-file/config handling and pySimPulseqSBB for the SWIG-wrapped C++ core) giving input/output-compatible parsing and simulation) — https://github.com/KerstinHut/pypulseq-cest
- **pulseq-cest-library** (Source of pre-defined, approved .seq saturation schemes and yaml simulation settings that serve as parsing inputs) — https://github.com/kherz/pulseq-cest-library

## Examples

```
seq = SequenceSBB.read('examples/APTw_3T_example.seq'); seq.plotSaturationPhase();
```

## Evaluation signals

- plotSaturationPhase() shows the expected RF magnitude, phase and offset scheduling for the loaded .seq file (e.g. the 100 ms HSExp pulse and 39 offsets over ±6 ppm in SLExp_3T_Phantom.seq)
- MATLAB and Python parses of the identical .seq file yield element-wise identical simulation inputs/outputs, as reported in Supporting Information Table S5
- The parsed event stream drives the Bloch-McConnell simulation to completion, producing one Z value per defined saturation offset with normalized Z values in the range 0–1
- In the interpreter context, the ADC-containing block is skipped and control is handed to the readout without errors

## Limitations

- The Pulseq SBB/interpreter is so far only available for Siemens scanners; parsing and playback for Philips, United or Canon requires further interpreter development
- Source code for the Siemens IDEA interpreter sequence is only available upon request, so parsed .seq files cannot be directly played out outside Siemens without such access
- The ADC event must not be combined with any other events in its block, since it is used solely as an internal marker to interrupt Pulseq playout

## Evidence

- [intro] Pulseq file definition: "In Pulseq, all sequence parameters are defined in a text-file (in the following termed as pulseq-file), which can be created with various popular programming applications"
- [methods] Events written and read by interpreter: "RF pulse, gradient, ADC and trigger events can be easily defined and are written to a pulseq-file, which is then read and played out by a native interpreter sequence"
- [methods] Simulation consumes parsed events: "performs Bloch-McConnell simulations for the respective sequence events37, and returns the current magnetization vector after preparation"
- [readme] MATLAB loading of seq-files: "All .seq-files can be loaded in MATLAB for plotting and detailed inspection."
- [readme] Python seq handling via BMCTool: [evidence span withheld — anchor status: not_found]
