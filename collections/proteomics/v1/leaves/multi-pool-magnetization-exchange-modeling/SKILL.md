---
name: multi-pool-magnetization-exchange-modeling
description: Use when a CEST .seq-file (e.g. SLExp_3T_Phantom.seq) must be simulated
  under a pool configuration from published SI tables (single amide pool or four D-glucose
  pools) to predict or reproduce Z-spectra.
when_to_use_negative:
- Bloch-McConnell based quantification is extremely challenging in vivo — do not expect
  reliable absolute pool quantification from in-vivo Z-spectra
- The task is to play out the preparation block on a scanner rather than simulate
  it — use the vendor-specific interpreter sequence instead
- The CEST protocol is not defined as a pulseq-file; non-standardized saturation definitions
  cannot be simulated with the pulseq-cest sim pipeline
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pulseq
  role: Open file format in which the CEST preparation period to be simulated is defined
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: MATLAB
  role: Environment for calling the C++ Bloch-McConnell mex-function and reading pulseq-files
    with SequenceSBB
  repo: https://github.com/kherz/pulseq-cest
- name: Python
  role: Environment for running the SWIG-wrapped pySimPulseqSBB simulation via pypulseq-cest
    and BMCTool
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: C++
  role: Core Bloch-McConnell solver (SimPulseqSBB) that loops through the pulseq-file
    and performs the simulations for the sequence events
  repo: https://github.com/kherz/pulseq-cest
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:1ce9e3160f3529373c8f753b6d83304dc46bd066fa3a95aae4ed85732dd92785
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - MATLAB
  - Pulseq
  - Python
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# multi-pool-magnetization-exchange-modeling

## Summary

Configure and run Bloch-McConnell simulations of CEST pulseq-files with pool-specific configurations (single amide pool or up to four solute pools such as D-glucose), returning Z-spectra whose shapes match measured data. The skill exists because the exact pulseq-defined saturation period plus flexible multi-pool simulation was shown to produce a good match between simulations and measurements.

## When to use

Use when a CEST .seq-file (e.g. SLExp_3T_Phantom.seq) must be simulated under a pool configuration from published SI tables (single amide pool or four D-glucose pools) to predict or reproduce Z-spectra.

## When NOT to use

- Bloch-McConnell based quantification is extremely challenging in vivo — do not expect reliable absolute pool quantification from in-vivo Z-spectra
- The task is to play out the preparation block on a scanner rather than simulate it — use the vendor-specific interpreter sequence instead
- The CEST protocol is not defined as a pulseq-file; non-standardized saturation definitions cannot be simulated with the pulseq-cest sim pipeline

## Inputs

- Pulseq .seq-file defining the CEST preparation block (e.g. SLExp_3T_Phantom.seq)
- YAML simulation config with pool definitions and settings from the pulseq-cest-library sim-library
- Pool configuration parameters from Supporting Information Tables S2-S4 (single amide pool or four D-glucose pools)

## Outputs

- Simulated Z-spectrum (water-pool z-magnetization vs frequency offset) per pool configuration
- Final magnetization vectors of the water pool (SimulationParameters object in Python)
- Comparison table of simulated vs measured Z-spectra normalized at −6 ppm

## How to apply

Load the pulseq-file with the pulseq-cest SequenceSBB reader and read the ROI-specific pool configurations from the Supporting Information tables (S2-S4). Configure the C++ Bloch-McConnell mex-function — called from MATLAB, or via the SWIG-wrapped pySimPulseqSBB in Python — for a single-amide-pool model (L-arginine ROIs 1-2) or a four-pool model (D-glucose ROI 3), pairing the .seq-file with a yaml config holding the simulation settings. Run the simulation over the full offset range (e.g. 39 offsets spanning ±6 ppm with a 100 ms HSExp spin-lock pulse and Trec = 5 s) and normalize the resulting Z-spectra by the measurement at −6 ppm for comparison against measured data. Pass-fail is judged by the match between simulated and measured Z-spectra, and runtime benchmarks (≈0.19 s per Z-spectrum for a single amide pool, ≈0.77 s for four pools, 500 samples per pulse on an Intel i7-7700K) can serve as sanity checks.

## Related tools

- **Pulseq** (Open file format in which the CEST preparation period to be simulated is defined) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **MATLAB** (Environment for calling the C++ Bloch-McConnell mex-function and reading pulseq-files with SequenceSBB) — https://github.com/kherz/pulseq-cest
- **Python** (Environment for running the SWIG-wrapped pySimPulseqSBB simulation via pypulseq-cest and BMCTool) — https://github.com/KerstinHut/pypulseq-cest
- **C++** (Core Bloch-McConnell solver (SimPulseqSBB) that loops through the pulseq-file and performs the simulations for the sequence events) — https://github.com/kherz/pulseq-cest

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq', normalize=True)
```

## Evaluation signals

- Measured per-Z-spectrum runtimes fall within a small tolerance of the reported ≈0.19 s (single amide pool) and ≈0.77 s (four-pool) benchmarks on comparable hardware (Intel i7-7700K, 500 samples per pulse)
- Simulated Z-spectra match measured phantom Z-spectra after normalization by the −6 ppm measurement
- Timing comparison table written to simulation_timing_report.csv with pass/fail per pool configuration
- Offsets used for simulation match the pulseq-file's defined offset grid (e.g. 39 offsets over ±6 ppm)

## Limitations

- Bloch-McConnell based quantification is extremely challenging in vivo
- Reported runtimes are hardware-dependent (reference: Intel i7-7700K Kaby Lake) and scale with the number of pools
- The simulation requires the .seq-file plus a yaml config; if the pool configuration is unknown, results are not interpretable against published measurements

## Evidence

- [methods] an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [supplementary] Simulating the pulseq-file took approximately 0.19 s per Z-spectrum for the single amide CEST pool: [evidence span withheld — anchor status: not_found]
- [abstract] The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [readme] In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings.: "In this folder you can find the  Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
- [methods] Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
