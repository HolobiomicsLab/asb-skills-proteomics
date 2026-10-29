---
name: intravoxel-phase-dispersion-simulation
description: Decision support for choosing allowable residual moment tolerances (M0/M1/M2)
  in motion-compensated diffusion gradient design, when you must trade optimizer compute
  time against ADC accuracy ≤5%.
when_to_use_negative:
- Input waveforms are already exactly moment-nulled (zero residual M0/M1/M2), so no
  phase-dispersion bias can arise to quantify
- The goal is optimizing readout trajectories or slice-select gradients — the paper
  states GrOpt 'has not been used to optimize readout trajectories or slice-select
  gradients'
- The analysis concerns eddy currents, PNS, or other constraints rather than residual
  moment–driven intravoxel phase dispersion
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3392
tools:
- name: GrOpt
  role: ADMM-based gradient waveform optimizer used to generate motion-compensated
    diffusion waveforms across residual moment tolerances
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Core optimization routines compiled as C kernels for the waveform generation
    feeding the simulation
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines for the Python wrapper used to drive the
    tolerance sweep
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper environment used to run the optimizer sweep, the phase-dispersion
    simulation, and result tabulation
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:46b214c5c57e2bf3902ebf793746b18d85a3c43db9736f9f5ac53b403b23f0cb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/cmr-group/gropt
  repo_url: https://github.com/cmr-group/gropt
  tools:
  - C
  - Cython
  - GrOpt
  - Python
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3392
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# intravoxel-phase-dispersion-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A numerical simulation workflow that quantifies how residual gradient moments (arising from non-zero ADMM convergence criteria in GrOpt-optimized diffusion waveforms) cause intravoxel phase dispersion, propagate into signal attenuation, and bias the apparent diffusion coefficient (ADC). It establishes the residual-moment tolerances (M0 ~1e-2 mT/m·ms, M1 ~1e-4 mT/m·ms²) at which ADC bias stays ≤5%.

## When to use

Decision support for choosing allowable residual moment tolerances (M0/M1/M2) in motion-compensated diffusion gradient design, when you must trade optimizer compute time against ADC accuracy ≤5%.

## When NOT to use

- Input waveforms are already exactly moment-nulled (zero residual M0/M1/M2), so no phase-dispersion bias can arise to quantify
- The goal is optimizing readout trajectories or slice-select gradients — the paper states GrOpt 'has not been used to optimize readout trajectories or slice-select gradients'
- The analysis concerns eddy currents, PNS, or other constraints rather than residual moment–driven intravoxel phase dispersion

## Inputs

- GrOpt-optimized motion-compensated diffusion encoding gradient waveforms generated across a sweep of allowed residual M0/M1/M2 moment tolerances
- Per-waveform achieved (residual, non-zero) gradient moment values and compute times from the ADMM optimizer
- Diffusion sequence parameters needed for the intravoxel phase dispersion / signal attenuation simulation

## Outputs

- Table of ADC bias and compute time versus allowed residual moment (M0/M1/M2) for every tolerance setting
- Summary figure showing isocontours of ADC bias as a function of residual gradient moments
- Estimated ADC values from simulated signals relative to a zero-residual reference

## How to apply

Build GrOpt from github.com/cmr-group/gropt by compiling the C kernels and Cython wrapper with 'python setup.py build_ext --inplace'. Use the ADMM optimizer (stopped on constraint feasibility and primal residual convergence, capped at 20,000 iterations) to generate motion-compensated diffusion waveforms over a sweep of allowed residual M0, M1, and M2 tolerances, recording compute time per setting. Compute the residual (non-zero) moments actually achieved by each waveform, then run the numerical simulation of intravoxel phase dispersion induced by each residual in the diffusion sequence, propagating phase spread into signal attenuation. Estimate ADC from the simulated signals and tabulate ADC bias (relative to a zero-residual reference) and compute time versus allowed residual moment, saving the table and a summary figure. Judge success by reproducing the paper's trade-off: larger allowed residual moments reduce iterations/compute time, while data accuracy remains acceptable (≤5% ADC increase) for small residuals — on the order of 10^-2 mT/m·ms for M0 or 10^-4 mT/m·ms² for M1.

## Related tools

- **GrOpt** (ADMM-based gradient waveform optimizer used to generate motion-compensated diffusion waveforms across residual moment tolerances) — https://github.com/cmr-group/gropt
- **C** (Core optimization routines compiled as C kernels for the waveform generation feeding the simulation) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines for the Python wrapper used to drive the tolerance sweep) — https://github.com/cmr-group/gropt
- **Python** (Wrapper environment used to run the optimizer sweep, the phase-dispersion simulation, and result tabulation) — https://github.com/cmr-group/gropt

## Examples

```
git clone https://github.com/cmr-group/gropt && cd gropt/python && python setup.py build_ext --inplace
```

## Evaluation signals

- Output table contains compute-time and ADC-accuracy entries for every residual-moment setting tested
- ADC bias stays ≤5% when residual M0 is on the order of 10^-2 mT/m·ms or residual M1 is on the order of 10^-4 mT/m·ms²
- Optimizer runs are capped at 20,000 iterations; settings that fail feasibility must be flagged, not silently accepted
- Achieved residual moments computed from the waveforms match the requested tolerance levels within the ADMM convergence criteria
- Reproduced trade-off: larger allowed residual moments reduce iterations/compute time while ADC remains acceptable for small residuals

## Limitations

- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof of convexity, so optimizer outputs may not be globally optimal
- If no feasible solution exists or cannot be found, the optimization is simply stopped after 20,000 iterations — the waveform and its residual moment may be unreliable
- Choice of N (gradient size) and dt (raster time) may need adjustment for longer gradient waveforms such as multiecho turbo-spin-echo sequences

## Evidence

- [methods] ADMM basis of GrOpt: [evidence span withheld — anchor status: not_found]
- [readme] Build instructions: [evidence span withheld — anchor status: external_doc]
