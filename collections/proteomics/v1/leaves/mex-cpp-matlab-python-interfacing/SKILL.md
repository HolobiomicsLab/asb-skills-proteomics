---
name: mex-cpp-matlab-python-interfacing
description: Use when a numerically intensive simulation kernel written in C++ must
  be exposed to both MATLAB and Python users without maintaining divergent re-implementations
  of the core algorithm.
when_to_use_negative:
- The target users need only one language front-end and a pure Python or MATLAB re-implementation
  would be simpler to maintain
- The algorithm is not performance-critical, so the mex/SWIG compilation toolchain
  overhead (compiler setup, platform-specific builds) is not justified
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pulseq-CEST
  role: Repository hosting the C++ Bloch-McConnell simulation core compiled as mex-function
    (MATLAB) and SWIG-wrapped package (Python)
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python front-end wrapping the C++ core via SWIG (pySimPulseqSBB) with parser
    functions (pypulseq_cest, BMCTool)
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: MATLAB
  role: Language front-end consuming the compiled mex-function to run the simulation
    on .seq-files (SequenceSBB.read)
- name: C++
  role: Native language of the shared simulation core that loops through pulseq-file
    sequence events and performs Bloch-McConnell simulations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:317049dd0af188afd2e4d54f98a00d4780a36ac2fe920e11f2d2e5da6f1d8f91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - MATLAB
  - Pulseq-CEST
  - Python
  - pypulseq-cest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mex-cpp-matlab-python-interfacing

## Summary

Compiles a shared C++ simulation core (here the Bloch-McConnell solver of pulseq-cest-sim) as a MATLAB mex-function and wraps the same code with SWIG for Python, so that MATLAB and Python give identical inputs and outputs from one native implementation.

## When to use

Use when a numerically intensive simulation kernel written in C++ must be exposed to both MATLAB and Python users without maintaining divergent re-implementations of the core algorithm.

## When NOT to use

- The target users need only one language front-end and a pure Python or MATLAB re-implementation would be simpler to maintain
- The algorithm is not performance-critical, so the mex/SWIG compilation toolchain overhead (compiler setup, platform-specific builds) is not justified

## Inputs

- C++ source files (e.g. BlochMcConnellSolver.h, SimPulseqSBB.cpp, SimulationParameters.cpp) with Eigen matrix library
- Pulseq .seq-file defining the CEST saturation block (e.g. examples/APTw_3T_example.seq)
- YAML simulation configuration file (tissue parameters: CEST pools, semi-solid MTC pool, Lorentzian or Super-Lorentzian line shape)
- SWIG interface files (pySimPulseqSBB.i, numpy.i, eigen.i) and setup.py for the Python build

## Outputs

- Compiled MATLAB mex-function exposing the Bloch-McConnell simulation
- pySimPulseqSBB Python package (pre-compiled distributions for Linux/Windows, Python 3.6-3.9)
- Per-offset magnetization vectors / Z-spectrum values from either front-end (e.g. written to simulated_z_spectrum.csv)

## How to apply

Write the core algorithm once in C++ (e.g. SimPulseqSBB with BlochMcConnellSolver.h), keeping Eigen for matrix handling. For the MATLAB path, compile the C++ code as a 64-bit mex-function (reference setup: Windows 10 with Microsoft Visual C++ 2017 compiler) that loops through the Pulseq .seq-file sequence events and returns the magnetization vector after the preparation period. For the Python path, expose the identical C++ classes through SWIG (.i files such as pySimPulseqSBB.i) to build the pySimPulseqSBB package, and ship pre-compiled distributions for Linux/Windows and Python 3.6-3.9 via setup.py; front-end parser packages (pypulseq_cest, BMCTool) then handle seq-file and yaml config handling. The rationale is that the same .seq file used on the scanner drives both language front-ends, guaranteeing identical simulation behavior and cross-language reproducibility.

## Related tools

- **Pulseq-CEST** (Repository hosting the C++ Bloch-McConnell simulation core compiled as mex-function (MATLAB) and SWIG-wrapped package (Python)) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python front-end wrapping the C++ core via SWIG (pySimPulseqSBB) with parser functions (pypulseq_cest, BMCTool)) — https://github.com/KerstinHut/pypulseq-cest
- **MATLAB** (Language front-end consuming the compiled mex-function to run the simulation on .seq-files (SequenceSBB.read))
- **C++** (Native language of the shared simulation core that loops through pulseq-file sequence events and performs Bloch-McConnell simulations)

## Examples

```
python setup.py  # compiles/wraps the C++ core as pySimPulseqSBB; then: from pypulseq_cest.simulate import sim_example; sim_example()
```

## Evaluation signals

- MATLAB and Python front-ends produce identical inputs/outputs when given the same .seq-file and yaml config
- simulateExample (MATLAB) and sim_example (Python) run to completion without error
- The output contains one Z value per defined saturation offset in the .seq-file
- All normalized Z values (normalized by the -6 ppm offset) fall in the range 0-1
- Pre-compiled pySimPulseqSBB distributions install cleanly via setup.py into a clean python/anaconda environment

## Limitations

- mex compilation requires a specific toolchain (64-bit Windows 10, Microsoft Visual C++ 2017 in the reference setup); compiler mismatches cause build failures
- Pre-compiled Python distributions only cover Python versions 3.6-3.9 on Linux and Windows; other versions require local compilation
- Python path needs additional dependencies (Git for installation, PyYAML, numpy, matplotlib) and the pulseq-cest-library must be present in the library subfolder

## Evidence

- [full_text] Compile the native C++ Bloch-McConnell simulation as a MATLAB mex-function (64-bit Windows 10, Microsoft Visual C++ 2017 compiler) and, for the Python path, wrap the same C++ core with SWIG behind the python parser so MATLAB and Python give identical inputs/outputs.: [evidence span withheld — anchor status: not_found]
- [methods] an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations"
- [readme] We provide several pre-compiled distributions of the C++ based simulation code for Linux and Windows and python versions between 3.6 and 3.9.: [evidence span withheld — anchor status: not_found]
- [readme] pySimPulseSBB: package containing the C++ based simulation code; pypulseq_cest: package containing the python based parser functions: [evidence span withheld — anchor status: not_found]
- [full_text] The pulseq-cest repository includes a Bloch-McConnell simulation (pulseq-cest-sim) that can be used to simulate and compare different Pulseq .seq-files: [evidence span withheld — anchor status: not_found]
