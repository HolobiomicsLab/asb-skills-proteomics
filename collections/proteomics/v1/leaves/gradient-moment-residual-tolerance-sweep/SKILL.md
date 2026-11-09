---
name: gradient-moment-residual-tolerance-sweep
description: Decision support for choosing motion-compensation convergence tolerances
  in optimized diffusion gradient waveforms when you need to know which residual M0/M1/M2
  magnitudes keep ADC bias within an acceptable bound (e.g., ≤5%).
when_to_use_negative:
- When waveforms must be exactly moment-nulled (zero convergence tolerance) rather
  than traded off against compute time — the sweep is specifically about quantifying
  acceptable residuals.
- When optimizing readout trajectories or slice-select gradients — the article states
  GrOpt has not been used for those and the simulations target diffusion encoding.
- When the input is measured imaging data rather than simulated signals — this skill
  quantifies ADC bias numerically, not from in-vivo scans.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: GrOpt
  role: ADMM-based optimizer generating motion-compensated diffusion waveforms across
    residual moment tolerance settings and recording convergence/compute time
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the core optimization routines compiled for the sweep
- name: Python
  role: Wrapper environment in which the tolerance sweep, phase-dispersion simulation,
    and ADC estimation are run
- name: Cython
  role: Builds the C kernels into the Python module via setup.py build_ext --inplace
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:f78463fd32032629c293224884a15c2eb245f3ad51597a8783ed6880b1fa2be4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/cmr-group/gropt
  tools:
  - C
  - Cython
  - GrOpt
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gradient-moment-residual-tolerance-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweeps allowed residual gradient moment (M0/M1/M2) tolerances in the GrOpt ADMM optimizer, then numerically simulates how the residual moments actually achieved by each waveform induce intravoxel phase dispersion and ADC bias in diffusion MRI. It quantifies the compute-time vs. data-accuracy trade-off of relaxing moment-nulling convergence criteria.

## When to use

Decision support for choosing motion-compensation convergence tolerances in optimized diffusion gradient waveforms when you need to know which residual M0/M1/M2 magnitudes keep ADC bias within an acceptable bound (e.g., ≤5%).

## When NOT to use

- When waveforms must be exactly moment-nulled (zero convergence tolerance) rather than traded off against compute time — the sweep is specifically about quantifying acceptable residuals.
- When optimizing readout trajectories or slice-select gradients — the article states GrOpt has not been used for those and the simulations target diffusion encoding.
- When the input is measured imaging data rather than simulated signals — this skill quantifies ADC bias numerically, not from in-vivo scans.

## Inputs

- GrOpt source tree (github.com/cmr-group/gropt) with C kernels in src/ and Python wrapper in python/
- Sweep specification of allowed residual gradient moment tolerances (M0, M1, M2) and gradient hardware/constraint limits defining the diffusion waveform
- Diffusion sequence parameters (b-value, timing, dt, N) and a zero-residual reference waveform for the ADC baseline

## Outputs

- Optimized diffusion-encoding gradient waveforms for each tolerance setting with their achieved residual M0/M1/M2 moments
- Table of compute time and ADC bias versus allowed residual moment level
- Summary figure (isocontours analogous to Figure 7A-C) mapping residual moments to ADC accuracy

## How to apply

Build GrOpt from github.com/cmr-group/gropt (compile the C kernels with the Cython wrapper via 'python setup.py build_ext --inplace' in the python/ directory). Using the ADMM-based optimizer — which stops on constraint feasibility and primal residual convergence and is hard-capped at 20,000 iterations — generate motion-compensated diffusion encoding waveforms across a sweep of allowed residual moment tolerances for M0, M1, and M2, recording compute time per setting. Compute the residual (non-zero) gradient moments actually achieved by each optimized waveform, then run a numerical simulation of the intravoxel phase dispersion each residual induces in the diffusion sequence, propagating phase spread into signal attenuation. Estimate the apparent diffusion coefficient (ADC) from the simulated signals and tabulate ADC bias relative to the zero-residual reference for each M0/M1/M2 level. The paper's isocontours (Figure 7A-C) show ≤5% ADC increase when residual M0 is on the order of 10^-2 mT/m×ms or residual M1 is on the order of 10^-4 mT/m×ms², while larger allowed residual moments reduce iterations and compute time (acceptable waveforms were generated in 1-40 ms).

## Related tools

- **GrOpt** (ADMM-based optimizer generating motion-compensated diffusion waveforms across residual moment tolerance settings and recording convergence/compute time) — github.com/cmr-group/gropt
- **C** (Language of the core optimization routines compiled for the sweep)
- **Python** (Wrapper environment in which the tolerance sweep, phase-dispersion simulation, and ADC estimation are run)
- **Cython** (Builds the C kernels into the Python module via setup.py build_ext --inplace)

## Examples

```
cd python && python setup.py build_ext --inplace
```

## Evaluation signals

- Output table contains compute-time and ADC-accuracy entries for every residual-moment setting in the sweep (no missing rows).
- ADC bias is ≤5% when residual M0 ≈ 10^-2 mT/m×ms or residual M1 ≈ 10^-4 mT/m×ms², matching the paper's Figure 7A-C isocontours.
- Larger allowed residual moments monotonically reduce iterations/compute time, reproducing the reported trade-off.
- Compute times for acceptable waveforms fall in the 1-40 ms range reported for GrOpt.
- Optimizer runs terminate either at constraint feasibility/primal residual convergence or at the 20,000-iteration cap.

## Limitations

- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof of convexity.
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive.
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences.
- If no feasible solution exists or cannot be found, the optimization stops after 20,000 iterations, so some tolerance settings may yield no valid waveform.

## Evidence

- [methods] ADMM basis of GrOpt: [evidence span withheld — anchor status: not_found]
- [readme] Python build instruction: [evidence span withheld — anchor status: external_doc]
