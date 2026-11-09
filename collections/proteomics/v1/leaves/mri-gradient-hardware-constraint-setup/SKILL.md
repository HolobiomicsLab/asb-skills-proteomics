---
name: mri-gradient-hardware-constraint-setup
description: Use when designing time-optimal MRI gradient waveforms (e.g., diffusion
  DWI, phase-contrast flow encoding) and you must specify hardware bounds gmax/smax,
  raster time dt, waveform size N, and moment/eddy-current constraints as GrOpt solver
  inputs.
when_to_use_negative:
- Optimizing readout trajectories or slice-select gradients — the article states GrOpt
  has not been used for these
- Problems requiring constraint types not yet in the toolbox, such as concomitant
  field terms, acoustic noise, or gradient heating limits
- Very long gradient waveforms (e.g., multiecho turbo-spin echo) where the default
  choice of N and dt may need adjustment beyond this skill's defaults
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3300
tools:
- name: GrOpt
  role: Optimization toolbox that consumes the gmax/smax/dt/moment constraint set
    and searches for a feasible time-optimal gradient waveform via ADMM
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the core optimization routines (src/optimize_kernel.c, minTE_diff)
    for native integration on scanner systems
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to call the C kernels (run_kernel_diff, minTE_diff) with the
    constraint parameters during prototyping
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines for the Python module, built with 'python
    setup.py build_ext --inplace'
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Alternative wrapper for prototyping and visualizing the constrained gradient
    waveforms
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:0c439b12d273ed765e88ad1841171802b39a98342a899f69276921a453f2f685
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
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mri-gradient-hardware-constraint-setup

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Translates MRI scanner gradient hardware limits (maximum gradient amplitude gmax, maximum slew rate smax, raster time dt) and imaging requirements (diffusion b-value, gradient moment nulling, eddy-current nulling, PNS limits) into the constraint set consumed by GrOpt's C optimization kernels so that generated waveforms are feasible on real hardware.

## When to use

Use when designing time-optimal MRI gradient waveforms (e.g., diffusion DWI, phase-contrast flow encoding) and you must specify hardware bounds gmax/smax, raster time dt, waveform size N, and moment/eddy-current constraints as GrOpt solver inputs.

## When NOT to use

- Optimizing readout trajectories or slice-select gradients — the article states GrOpt has not been used for these
- Problems requiring constraint types not yet in the toolbox, such as concomitant field terms, acoustic noise, or gradient heating limits
- Very long gradient waveforms (e.g., multiecho turbo-spin echo) where the default choice of N and dt may need adjustment beyond this skill's defaults

## Inputs

- Scanner gradient hardware limits: maximum gradient amplitude gmax and maximum slew rate smax
- Gradient waveform raster time dt and number of waveform points N
- Gradient moment nulling specification (moments_params, 7-value format; order M→n,mTmsn)
- Diffusion encoding target b-value (e.g., b = 1000 mm2/s) for minTE_diff/b-value search
- Optional eddy-current time constants to null, PNS model choice (Schulte/Hebrank), and slice obliquity angles

## Outputs

- A feasible time-optimal gradient waveform g(t) on the dt grid satisfying all constraints within tolerance
- Minimum feasible waveform duration T = dt*N (e.g., minimum TE for the target b-value)
- Infeasibility flag after the 20,000-iteration stopping rule if no solution is found

## How to apply

Build the GrOpt C kernels (src/optimize_kernel.c) and the Cython Python wrapper via 'python setup.py build_ext --inplace'. Formulate the inner constrained waveform problem by passing gmax, smax, fixed N and raster time dt, and gradient moment constraints in the 7-value moments_params format to run_kernel_diff() or minTE_diff(); each constraint c(g) is enforced against a bound bi within a specified tolerance, and moments of any order M→n,mTmsn can be nulled fully or partially to suppress motion error. Decide which additional modular filters apply: eddy-current nulling for specified time constants, PNS limits via the Schulte et al or Hebrank et al models, diffusion b-value maximization, or duty-cycle minimization. Handle slice obliquity by either the 'rotationally invariant' approach (derating hardware limits by a factor of 3) or the 'rotationally variant' approach that rotates the relevant constraints before solving. If no feasible waveform exists or is found within the tolerance, the optimization stops after 20,000 iterations, so review constraint feasibility (e.g., relax moments or extend duration T) rather than accepting an infeasible result.

## Related tools

- **GrOpt** (Optimization toolbox that consumes the gmax/smax/dt/moment constraint set and searches for a feasible time-optimal gradient waveform via ADMM) — https://github.com/cmr-group/gropt
- **C** (Language of the core optimization routines (src/optimize_kernel.c, minTE_diff) for native integration on scanner systems) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used to call the C kernels (run_kernel_diff, minTE_diff) with the constraint parameters during prototyping) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines for the Python module, built with 'python setup.py build_ext --inplace') — https://github.com/cmr-group/gropt
- **Matlab** (Alternative wrapper for prototyping and visualizing the constrained gradient waveforms) — https://github.com/cmr-group/gropt

## Examples

```
python setup.py build_ext --inplace  # then call run_kernel_diff()/minTE_diff() with gmax, smax, dt, N and moments_params from the Python wrapper
```

## Evaluation signals

- Every returned waveform satisfies all constraints c_i(g) within the specified tolerance against bounds b_i (e.g., gmax and smax never exceeded)
- The b-value of a diffusion waveform meets the requested target (e.g., b = 1000 mm2/s) at the reported minimum TE
- Infeasible problems terminate after at most 20,000 iterations rather than looping indefinitely
- For N > 64 waveform points, minimum TEs are within 1 ms of the shortest (time-optimal) TE; within 0.5 ms for N = 128
- Waveforms generated on different scanners from the same constraints are functionally equivalent with comparable images

## Limitations

- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Residual gradient moments due to non-zero convergence criteria affect intravoxel phase dispersion and ADC, so moment constraints must be verified numerically

## Evidence

- [methods] Slice obliquity is handled using either a "rotationally invariant" approach that derates the hardware limitations by 3: [evidence span withheld — anchor status: not_found]
- [readme] The Python module has been tested primarily with Anaconda and Python 3.7, though it should work with any type of Python environment.: [evidence span withheld — anchor status: not_found]
