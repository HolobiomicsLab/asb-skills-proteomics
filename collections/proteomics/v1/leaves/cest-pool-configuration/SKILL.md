---
name: cest-pool-configuration
description: Use when simulating CEST Z-spectra from pulseq .seq-files and a pool
  configuration (CEST pools, semi-solid MTC pool, line shape) must be specified in
  a yaml config before running the Bloch-McConnell simulation.
when_to_use_negative:
- When the goal is to define the saturation block itself (RF pulses, offsets, timing)
  rather than the simulation pool parameters — that belongs in the .seq-file, not
  the yaml config
- When quantifying exchange in vivo — the article notes Bloch-McConnell based quantification
  is extremely challenging in vivo, so pool configurations should be treated as simulation
  assumptions, not validated in vivo estimates
- When a vendor-specific scanner interpreter setup is needed; pool configuration is
  purely an in silico simulation concern
edam_operation: http://edamontology.org/operation_0244
tools:
- name: Pulseq-CEST
  role: Repository containing the Bloch-McConnell simulation (pulseq-cest-sim) that
    consumes the pool configuration and .seq-file
  repo: https://github.com/kherz/pulseq-cest
- name: pulseq-cest-library
  role: Provides pre-defined and approved simulation settings (yaml) in the sim-library
    to pair with seq-library .seq-files
  repo: https://github.com/kherz/pulseq-cest-library
- name: pypulseq-cest
  role: 'Python route: pySimPulseqSBB (SWIG-wrapped C++) runs the pool configuration;
    BMCTool handles yaml/seq file handling'
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: yamlmatlab
  role: Reads the yaml pool-configuration file into MATLAB for the MATLAB simulation
    route
  repo: https://github.com/ewiger/yamlmatlab
- name: Pulseq
  role: Open file format of the .seq-file the pool configuration parameterizes the
    simulation for
  repo: https://github.com/pulseq/pulseq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:8282eba4976166dbef820513506d5d405a02449f6a551e2acafe08b169972eb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - Pulseq
  - Pulseq-CEST
  - pulseq-cest-library
  - pypulseq-cest
  - yamlmatlab
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cest-pool-configuration

## Summary

Define the magnetization-exchange pool configuration (an arbitrary number of CEST pools plus one semi-solid MTC pool with a Lorentzian or Super-Lorentzian line shape) in a yaml file that parameterizes the pulseq-cest Bloch-McConnell simulation of .seq-file CEST preparation blocks.

## When to use

Use when simulating CEST Z-spectra from pulseq .seq-files and a pool configuration (CEST pools, semi-solid MTC pool, line shape) must be specified in a yaml config before running the Bloch-McConnell simulation.

## When NOT to use

- When the goal is to define the saturation block itself (RF pulses, offsets, timing) rather than the simulation pool parameters — that belongs in the .seq-file, not the yaml config
- When quantifying exchange in vivo — the article notes Bloch-McConnell based quantification is extremely challenging in vivo, so pool configurations should be treated as simulation assumptions, not validated in vivo estimates
- When a vendor-specific scanner interpreter setup is needed; pool configuration is purely an in silico simulation concern

## Inputs

- yaml simulation config file (e.g. config_example.yaml from pypulseq_cest/example_library or a sim-library entry) defining CEST pools and one semi-solid MTC pool (Lorentzian or Super-Lorentzian)
- pulseq .seq-file defining the CEST pre-saturation block (e.g. examples/APTw_3T_example.seq)

## Outputs

- Pool-parameterized Bloch-McConnell simulation producing per-offset magnetization vectors / Z-spectrum values
- SimulationParameters object (Python) exposing e.g. GetFinalMagnetizationVectors() for downstream Z-spectrum extraction

## How to apply

Create or select a yaml configuration file defining all simulation settings, including an arbitrary number of CEST pools plus one semi-solid MTC pool whose line shape is either Lorentzian or Super-Lorentzian, with relaxation and exchange parameters for each pool. Pair the yaml config with the pulseq .seq-file that defines the pre-saturation block; pre-defined and approved configs are available in the pulseq-cest-library sim-library. Feed both files to the C++-based SimPulseqSBB simulation — via the MATLAB compiled mex-function or the SWIG-wrapped Python package pySimPulseqSBB — which loops through the pulseq-file sequence events and performs Bloch-McConnell simulations, returning the magnetization vector of the pools after preparation. To replicate published settings, load the matching yaml file from the sim-library (in MATLAB via yamlmatlab's yaml.ReadYaml, in Python via PyYAML/BMCTool) rather than hand-editing pool parameters. The same pool configuration must be used identically across MATLAB and Python runs if the goal is cross-implementation comparison of the resulting Z-spectra.

## Related tools

- **Pulseq-CEST** (Repository containing the Bloch-McConnell simulation (pulseq-cest-sim) that consumes the pool configuration and .seq-file) — https://github.com/kherz/pulseq-cest
- **pulseq-cest-library** (Provides pre-defined and approved simulation settings (yaml) in the sim-library to pair with seq-library .seq-files) — https://github.com/kherz/pulseq-cest-library
- **pypulseq-cest** (Python route: pySimPulseqSBB (SWIG-wrapped C++) runs the pool configuration; BMCTool handles yaml/seq file handling) — https://github.com/KerstinHut/pypulseq-cest
- **yamlmatlab** (Reads the yaml pool-configuration file into MATLAB for the MATLAB simulation route) — https://github.com/ewiger/yamlmatlab
- **Pulseq** (Open file format of the .seq-file the pool configuration parameterizes the simulation for) — https://github.com/pulseq/pulseq

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq')
```

## Evaluation signals

- MATLAB and Python simulations run on the identical .seq-file and identical yaml pool configuration produce element-wise identical Z-spectrum values (agreement as reported in Supporting Information Table S5)
- The pool configuration contains an arbitrary number of CEST pools plus exactly one semi-solid MTC pool, with line shape set to Lorentzian or Super-Lorentzian
- The yaml file loads without error via yaml.ReadYaml (MATLAB) or PyYAML/BMCTool (Python) and its keys match the sim-library schema
- Normalized Z-spectra (normalized by the measurement at −6 ppm) show a plausible CEST contrast pattern for the chosen pools, e.g. APTw reduction near 3.5 ppm
- The simulation runs through the compiled C++ solver (mex-function in MATLAB, pySimPulseqSBB in Python) and returns magnetization vectors for every offset in the .seq-file

## Limitations

- Bloch-McConnell based quantification is extremely challenging in vivo, so pool configurations are simulation assumptions rather than measured tissue parameters
- Pre-defined pool settings must match the installed pulseq-cest-library version; installation instructions require downloading the library as .zip and specific Pulseq versions (1.3 or commit ed8cf4231dc7a5caaf828211486287afcdbac230)
- Cross-implementation equality is only guaranteed when the same .seq-file and same pool configuration are supplied to both the MATLAB and Python implementations

## Evidence

- [supplementary] Define the pool configuration for the simulation (an arbitrary number of CEST pools plus one semi-solid MTC pool with either a Lorentzian or Super-Lorentzian line shape).: [evidence span withheld — anchor status: not_found]
- [readme] All simulations require a yaml file with all simulation settings and a seq file with the pre-saturation block.: [evidence span withheld — anchor status: not_found]
- [readme] The sim-library contains pre-defined and approved simulation settings in yaml format.: [evidence span withheld — anchor status: not_found]
- [methods] C++ application loops through pulseq-files and performs Bloch-McConnell simulations for the sequence events.: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
