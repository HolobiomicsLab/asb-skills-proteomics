---
name: simulation-runtime-benchmarking
description: Use when reproducing or validating the computational performance of a
  CEST Bloch-McConnell simulation (e.g. pulseq-cest-sim) against published per-Z-spectrum
  runtime benchmarks.
when_to_use_negative:
- The goal is simulation accuracy (Z-spectrum values) rather than computational runtime
  — use the standard simulation validation workflow instead
- The sequence uses a different pulse or offset scheme than the 39-offset ±6 ppm 100
  ms HSExp spin-lock setup, so the published runtime benchmarks are not comparable
- Hardware differs substantially from the reference spec (Intel i7-7700K Kaby Lake)
  without re-baselining, making direct pass/fail comparison misleading
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pulseq
  role: File format of the .seq phantom sequence being simulated and timed
  repo: https://pulseq.github.io/
- name: Pulseq-CEST
  role: Provides SequenceSBB reader, Bloch-McConnell simulation, and seq/sim libraries
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python route via the SWIG-wrapped pySimPulseqSBB C++ simulation code
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: MATLAB
  role: Runs the C++ Bloch-McConnell mex-function for the timing benchmark
- name: C++
  role: Bloch-McConnell simulation engine whose per-Z-spectrum runtime is benchmarked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:bae514e891f71383ea40ce24eccd27b6cdfdb55a91012100b62015cc030894af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - MATLAB
  - Pulseq
  - Pulseq-CEST
  - pypulseq-cest
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# simulation-runtime-benchmarking

## Summary

Benchmarks the wall-clock runtime of the pulseq-cest Bloch-McConnell simulation per Z-spectrum for different pool configurations, using the reported ~0.19 s (single amide pool) and ~0.77 s (four D-glucose pools) figures as reference targets.

## When to use

Use when reproducing or validating the computational performance of a CEST Bloch-McConnell simulation (e.g. pulseq-cest-sim) against published per-Z-spectrum runtime benchmarks.

## When NOT to use

- The goal is simulation accuracy (Z-spectrum values) rather than computational runtime — use the standard simulation validation workflow instead
- The sequence uses a different pulse or offset scheme than the 39-offset ±6 ppm 100 ms HSExp spin-lock setup, so the published runtime benchmarks are not comparable
- Hardware differs substantially from the reference spec (Intel i7-7700K Kaby Lake) without re-baselining, making direct pass/fail comparison misleading

## Inputs

- Phantom pulseq-file (e.g. SLExp_3T_Phantom.seq) from the pulseq-cest-library seq-library
- ROI-specific pool configuration yaml files from Supporting Information Tables S2-S4 (single amide pool; four D-glucose CEST pools)
- Simulation settings yaml (sim-library) specifying offsets, pulse, and Trec

## Outputs

- Per-Z-spectrum wall-clock runtime measurements for each pool configuration (single-amide and four-pool)
- simulation_timing_report.csv comparing measured runtimes against the 0.19 s and 0.77 s benchmarks with pass/fail flags

## How to apply

Load the phantom pulseq-file (e.g. SLExp_3T_Phantom.seq) with the pulseq-cest SequenceSBB reader and read the ROI-specific pool configurations from Supporting Information Tables S2-S4. Configure the C++ Bloch-McConnell solver (MATLAB mex-function or the SWIG-wrapped Python pySimPulseqSBB) for the single-amide-pool model of the L-arginine ROIs 1-2 and the four-pool model for the D-glucose ROI 3. Run one full Z-spectrum per ROI (39 offsets over ±6 ppm with a 100 ms HSExp spin-lock pulse, Trec = 5 s, 500 samples per pulse) while measuring wall-clock time per spectrum. Repeat the timing for both pool configurations on comparable hardware (reference: Intel i7-7700K Kaby Lake) and compare against the approximate reported runtimes of 0.19 s (single pool) and 0.77 s (four-pool), flagging pass/fail per case and writing a comparison table to simulation_timing_report.csv.

## Related tools

- **Pulseq** (File format of the .seq phantom sequence being simulated and timed) — https://pulseq.github.io/
- **Pulseq-CEST** (Provides SequenceSBB reader, Bloch-McConnell simulation, and seq/sim libraries) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python route via the SWIG-wrapped pySimPulseqSBB C++ simulation code) — https://github.com/KerstinHut/pypulseq-cest
- **MATLAB** (Runs the C++ Bloch-McConnell mex-function for the timing benchmark)
- **C++** (Bloch-McConnell simulation engine whose per-Z-spectrum runtime is benchmarked)

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='config_SLExp_3T_Phantom.yaml', seq_file='SLExp_3T_Phantom.seq', normalize=True)
```

## Evaluation signals

- Measured runtime per Z-spectrum for the single-amide-pool configuration falls within a small tolerance of ~0.19 s
- Measured runtime per Z-spectrum for the four-pool D-glucose configuration falls within a small tolerance of ~0.77 s
- The four-pool runtime is roughly 4x the single-pool runtime, consistent with pool-count scaling
- simulation_timing_report.csv is written with both benchmark cases and explicit pass/fail flags
- Simulation used 500 samples per pulse, 39 offsets over ±6 ppm, 100 ms HSExp pulse, and Trec = 5 s matching the reference setup

## Limitations

- Runtimes are approximate and hardware-dependent; the reported benchmarks were obtained on an Intel i7-7700K Kaby Lake CPU, so absolute times will differ on other machines
- The pulseq-cest Bloch-McConnell simulation benchmarks apply to the phantom pool configurations (single amide; four D-glucose pools); other pool models or sequences need fresh baselining
- Runtime is reported per Z-spectrum for the specific 39-offset ±6 ppm protocol; sequences with different offsets, samples per pulse, or Trec will scale differently

## Evidence

- [results] Simulating the pulseq-file took approximately 0.19 s per Z-spectrum for the single amide CEST pool (ROIs 1 and 2) and 0.77 s for four different D-glucose CEST pools (ROI 3), using 500 samples per pulse.: [evidence span withheld — anchor status: not_found]
- [methods] an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [readme] In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings.: "In this folder you can find the  Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
- [methods] A phantom was prepared using multiple 6 ml tubes with either L-arginine or D-glucose: "A phantom was prepared using multiple 6 ml tubes with either L-arginine or D-glucose"
