---
name: cross-language-numerical-validation
description: Use when the same computational model exists in two languages (e.g.,
  MATLAB and Python) and you must prove the implementations agree on identical inputs
  before trusting either output.
when_to_use_negative:
- The two implementations use different input files or pool configurations — differences
  then reflect input mismatch, not implementation disagreement
- The goal is validating simulator accuracy against measured data rather than agreement
  between implementations (use scanner phantom measurements for that)
- The model has language-specific floating-point or initialization settings that are
  not aligned, making element-wise equality an inappropriate criterion
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pulseq-CEST
  role: MATLAB-side Bloch-McConnell simulation (SequenceSBB, compiled C++ mex-function)
    that simulates the .seq-file and pool configuration
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: 'Python-side simulation: C++ code wrapped with SWIG (pySimPulseqSBB) plus
    a python parser package, run on the identical .seq-file and YAML config'
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: Pulseq
  role: Open, vendor-independent file format that provides the shareable .seq-file
    used as the identical input for both implementations
  repo: https://github.com/pulseq/pulseq
- name: MATLAB
  role: Environment for loading the .seq-file with the SequenceSBB class and running
    the MATLAB simulation path
- name: Python
  role: Environment for the pypulseq_cest.simulate.simulate call and downstream Z-spectrum
    extraction (get_zspec)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:49e978b9969b24bc82e3d8d08c8090a49742708c96772865be862ce6e3afcefe
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
  - pypulseq-cest
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Cross-language numerical validation

## Summary

Validates that two independent implementations of the same numerical model (here: MATLAB and Python Bloch-McConnell simulations of CEST preparation blocks defined in Pulseq .seq-files) produce element-wise identical outputs, ensuring input/output compatibility and reproducibility across language ecosystems.

## When to use

Use when the same computational model exists in two languages (e.g., MATLAB and Python) and you must prove the implementations agree on identical inputs before trusting either output.

## When NOT to use

- The two implementations use different input files or pool configurations — differences then reflect input mismatch, not implementation disagreement
- The goal is validating simulator accuracy against measured data rather than agreement between implementations (use scanner phantom measurements for that)
- The model has language-specific floating-point or initialization settings that are not aligned, making element-wise equality an inappropriate criterion

## Inputs

- Pulseq .seq-file defining the CEST saturation preparation block (e.g., examples/APTw_3T_example.seq)
- Pool configuration file in YAML format (CEST pools + one semi-solid MTC pool, Lorentzian or Super-Lorentzian line shape), shared by both implementations
- MATLAB installation with pulseq-cest (SequenceSBB class, compiled C++ mex-function)
- Python environment with pySimPulseqSBB (SWIG-wrapped C++), pypulseq_cest, and BMCTool packages

## Outputs

- MATLAB-simulated Z-spectrum (magnetization vector after preparation per offset)
- Python-simulated Z-spectrum on the identical inputs
- Element-wise equality-check results confirming input/output compatibility (Supporting Information Table S5)
- cross_implementation_comparison.csv containing the cross-implementation Z-spectrum values and equality results

## How to apply

Define a fixed input contract — here the same Pulseq .seq-file (e.g., examples/APTw_3T_example.seq loaded via the SequenceSBB class in MATLAB) and the identical pool configuration (an arbitrary number of CEST pools plus one semi-solid MTC pool with Lorentzian or Super-Lorentzian line shape) in YAML format. Run the MATLAB Bloch-McConnell simulation via the compiled C++ mex-function on the .seq-file and pool configuration, then run the Python implementation (C++ code wrapped with SWIG and simplified by a python parser, e.g. pySimPulseqSBB via pypulseq_cest.simulate.simulate) on the identical inputs. Compare the two Z-spectrum outputs element-wise, normalizing both by the same reference (e.g., the measurement at −6 ppm) if normalization applies. Record the per-element equality-check results (agreement reported in Supporting Information Table S5) and write them, together with the cross-implementation Z-spectrum values, to a shared output file such as cross_implementation_comparison.csv.

## Related tools

- **Pulseq-CEST** (MATLAB-side Bloch-McConnell simulation (SequenceSBB, compiled C++ mex-function) that simulates the .seq-file and pool configuration) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python-side simulation: C++ code wrapped with SWIG (pySimPulseqSBB) plus a python parser package, run on the identical .seq-file and YAML config) — https://github.com/KerstinHut/pypulseq-cest
- **Pulseq** (Open, vendor-independent file format that provides the shareable .seq-file used as the identical input for both implementations) — https://github.com/pulseq/pulseq
- **MATLAB** (Environment for loading the .seq-file with the SequenceSBB class and running the MATLAB simulation path)
- **Python** (Environment for the pypulseq_cest.simulate.simulate call and downstream Z-spectrum extraction (get_zspec))

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq', normalize=True)
```

## Evaluation signals

- Element-wise comparison of MATLAB and Python Z-spectrum values shows agreement (as reported in Supporting Information Table S5)
- Both runs consumed byte-identical inputs: the same .seq-file and the same YAML pool configuration from the pulseq-cest-library sim-library
- Extracted offsets and Z-magnetization values from the Python SimulationParameters object (get_zspec) match the MATLAB output offsets
- Both Z-spectra are normalized consistently (e.g., by the −6 ppm measurement) before comparison
- Cross-implementation values and equality-check results are present and complete in cross_implementation_comparison.csv

## Limitations

- The Python installation relies on pre-compiled C++ distributions for Linux/Windows and Python versions 3.6–3.9; other environments may fail to build
- Element-wise agreement checks equivalence of implementations, not correctness against measured in-vivo data; Bloch-McConnell based quantification is extremely challenging in vivo
- Simulation settings and approved pre-saturation schemes must come from the pulseq-cest-library (seq-library and sim-library); ad-hoc configs may not be compatible with both parsers

## Evidence

- [intro] The pulseq-cest repository includes a Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings.: "In this folder you can find the Bloch-McConnell simulation that can be used to simulate and compare different .seq-files for different settings."
- [readme] Python implementation is C++ wrapped with SWIG plus a python parser: "|     | -- compile (C++, SWIG): C++ code for simulation handling (SimPulseqSBB), 
|     |     |                   SWIG for direct wrapping (pySimPulseqSBB)"
- [methods] C++ application performs Bloch-McConnell simulations of the sequence events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [abstract] Pulseq-CEST provides a straightforward approach to standardize, share, simulate, and measure different CEST preparation schemes: "With Pulseq-CEST, we provide a straightforward approach to standardize, share, simulate, and measure different CEST preparation schemes"
- [methods] Z-spectra normalized by the measurement at −6 ppm before comparison: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [readme] Library holds shared pre-defined schemes and simulation settings for both implementations: "It consists of a [seq-library](https://github.com/kherz/pulseq-cest-library/tree/master/seq-library) with several 
pre-defined and approved pre-saturation schemes in the"
