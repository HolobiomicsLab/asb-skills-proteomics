---
name: z-spectrum-simulation
description: Use when a CEST saturation protocol is defined as a Pulseq .seq-file
  and an in-silico Z-spectrum is needed to test, optimize, or cross-validate the preparation
  block before or instead of scanner measurement.
when_to_use_negative:
- The task is quantification of absolute metabolite concentrations in vivo — Bloch-McConnell
  based quantification is described as extremely challenging in vivo and is beyond
  this skill's scope.
- The CEST preparation is not available in the open Pulseq format (e.g. a proprietary
  vendor-only implementation such as the Siemens IDEA interpreter whose source is
  only available upon request).
- Only a measured Z-spectrum is needed for routine MTR_asym analysis without any in-silico
  modeling of the preparation block.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: Pulseq-CEST
  role: 'MATLAB-side framework: SequenceSBB reader loads the .seq-file and invokes
    the compiled C++ Bloch-McConnell mex-function to simulate the preparation period'
  repo: https://github.com/kherz/pulseq-cest
- name: PyPulseq-CEST
  role: 'Python-side implementation: SWIG-wrapped C++ simulation core (pySimPulseSBB)
    with parser functions, driven by a yaml config and .seq file via the simulate()
    function'
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: Pulseq
  role: Defines the open file format of the .seq-file that both the scanner interpreter
    and the simulation read
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: pulseq-cest-library
  role: Source of pre-defined, approved .seq saturation schemes (seq-library) and
    simulation settings in yaml format (sim-library)
  repo: https://github.com/kherz/pulseq-cest-library
- name: yamlmatlab
  role: MATLAB dependency used to read the yaml simulation configuration files
  repo: https://github.com/ewiger/yamlmatlab
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:41c0b664f3f008d3cfbec6820e13e03519b5fc98b01c1be2ffc57921bdd8573c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - Pulseq
  - Pulseq-CEST
  - PyPulseq-CEST
  - pulseq-cest-library
  - yamlmatlab
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# z-spectrum-simulation

## Summary

Simulates the Z-spectrum of a CEST preparation period defined in a Pulseq .seq-file using the open-source Bloch-McConnell simulation (pulseq-cest-sim), runnable from both MATLAB (compiled C++ mex-function) and Python (same C++ core wrapped with SWIG behind a python parser), producing identical magnetization vectors per saturation offset.

## When to use

Use when a CEST saturation protocol is defined as a Pulseq .seq-file and an in-silico Z-spectrum is needed to test, optimize, or cross-validate the preparation block before or instead of scanner measurement.

## When NOT to use

- The task is quantification of absolute metabolite concentrations in vivo — Bloch-McConnell based quantification is described as extremely challenging in vivo and is beyond this skill's scope.
- The CEST preparation is not available in the open Pulseq format (e.g. a proprietary vendor-only implementation such as the Siemens IDEA interpreter whose source is only available upon request).
- Only a measured Z-spectrum is needed for routine MTR_asym analysis without any in-silico modeling of the preparation block.

## Inputs

- Pulseq .seq-file defining the CEST preparation block (e.g. examples/APTw_3T_example.seq, SLExp_3T_Phantom.seq)
- YAML pool/simulation configuration file (one or more CEST pools plus one semi-solid MTC pool, Lorentzian or Super-Lorentzian line shape) from the pulseq-cest-library sim-library
- Saturation offset grid (e.g. 39 offsets over +/-6 ppm for the phantom protocol)

## Outputs

- Simulated Z-spectrum: one normalized Z value per defined saturation offset (CSV, e.g. simulated_z_spectrum.csv)
- Final magnetization vectors of the water pool (SimulationParameters object, sim.GetFinalMagnetizationVectors())
- Cross-implementation comparison table of MATLAB vs Python Z-spectrum values (cross_implementation_comparison.csv)

## How to apply

Load the .seq-file (e.g. examples/APTw_3T_example.seq) with the SequenceSBB class in MATLAB or via the BMCTool/pypulseq_cest parser in Python. Define a pool configuration YAML containing an arbitrary number of CEST pools plus one semi-solid MTC pool with either a Lorentzian or Super-Lorentzian line shape. Run the C++ Bloch-McConnell solver (SimPulseqSBB), which loops through the sequence events of the pulseq-file and returns the magnetization vector after the preparation period for each saturation offset. Normalize the simulated Z-spectrum by the value at the -6 ppm offset before comparison to measured data; MATLAB and Python implementations can be compared element-wise to confirm input/output compatibility. Validate that the simulation completes without error, produces one Z value per defined offset, and that normalized Z values fall in the 0-1 range.

## Related tools

- **Pulseq-CEST** (MATLAB-side framework: SequenceSBB reader loads the .seq-file and invokes the compiled C++ Bloch-McConnell mex-function to simulate the preparation period) — https://github.com/kherz/pulseq-cest
- **PyPulseq-CEST** (Python-side implementation: SWIG-wrapped C++ simulation core (pySimPulseSBB) with parser functions, driven by a yaml config and .seq file via the simulate() function) — https://github.com/KerstinHut/pypulseq-cest
- **Pulseq** (Defines the open file format of the .seq-file that both the scanner interpreter and the simulation read) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **pulseq-cest-library** (Source of pre-defined, approved .seq saturation schemes (seq-library) and simulation settings in yaml format (sim-library)) — https://github.com/kherz/pulseq-cest-library
- **yamlmatlab** (MATLAB dependency used to read the yaml simulation configuration files) — https://github.com/ewiger/yamlmatlab

## Examples

```
from pypulseq_cest.simulate import sim_example; sim_example()
```

## Evaluation signals

- simulateExample() (MATLAB) or sim_example() (Python) runs to completion without error
- Output file exists with exactly one Z value per defined saturation offset (e.g. 39 offsets for the +/-6 ppm phantom protocol)
- All normalized Z values fall in the range 0-1
- MATLAB and Python outputs match element-wise for the identical .seq-file and pool configuration, as reported in Supporting Information Table S5
- Measured runtimes are consistent with reported benchmarks (~0.19 s per Z-spectrum for a single amide pool, ~0.77 s for four D-glucose pools, 500 samples per pulse, Intel i7-7700K-class hardware)

## Limitations

- The C++ mex-function must be compiled for the platform (reference build: 64-bit Windows 10, Microsoft Visual C++ 2017 compiler); Python requires pre-compiled distributions for python 3.6-3.9 on Linux or Windows.
- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for Philips, United, or Canon is needed for universal application, so scanner-played output for cross-checking is vendor-limited.
- Tissue model is restricted to CEST pools plus one semi-solid MTC pool with Lorentzian or Super-Lorentzian line shape; Bloch-McConnell based quantification is extremely challenging in vivo.
- Reported runtimes depend on hardware; the 0.19 s / 0.77 s per-Z-spectrum benchmarks are for 500 samples per pulse on an Intel i7-7700K.

## Evidence

- [readme] In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings.: "In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
- [methods] an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [abstract] The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [intro] a fast and flexible open source simulation for the same pulseq-files that are played out on the MR scanner: "we present a fast and flexible open source simulation for the same pulseq-files that are played out on the MR scanner"
