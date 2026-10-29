---
name: mri-gradient-waveform-optimization
description: Use when a pulse sequence needs a gradient waveform generated on-the-fly
  (within ~1-40 ms) subject to hardware limits (Gmax, Smax), gradient moment nulling
  (M0/M1/M2), eddy-current nulling, or PNS constraints.
when_to_use_negative:
- The task is optimizing readout trajectories or slice-select gradients — the article
  states GrOpt has not been used for these
- A constraint on concomitant field terms, acoustic noise, or gradient heating limits
  is required — these are listed as future work not yet included
- Off-the-shelf vendor waveforms suffice and no arbitrary motion compensation, eddy-current
  nulling, or PNS constraint is needed
edam_operation: http://edamontology.org/operation_3927
tools:
- name: GrOpt
  role: Core ADMM-based optimizer solving the constrained waveform feasibility problem
    and outer time-optimal duration search
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the optimization kernels in src/optimize_kernel.c for native integration
    in pulse sequences
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C kernels to build the importable Python module via setup.py
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper for prototyping and visualizing gradient waveforms (Jupyter notebook
    demos in ./python/)
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  - task_001
  - task_008
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:e3ab88faec03b6c68b7af80ed2dd6fe31693e700d2b30cfa2a3abc444ab68010
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mri-gradient-waveform-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for rapidly designing time-optimal MRI gradient waveforms (diffusion, phase-contrast, flow-compensated) via the open-source GrOpt toolbox, which solves constraint-based feasibility problems using an ADMM optimizer with a multi-threaded outer search over waveform duration T.

## When to use

Use when a pulse sequence needs a gradient waveform generated on-the-fly (within ~1-40 ms) subject to hardware limits (Gmax, Smax), gradient moment nulling (M0/M1/M2), eddy-current nulling, or PNS constraints.

## When NOT to use

- The task is optimizing readout trajectories or slice-select gradients — the article states GrOpt has not been used for these
- A constraint on concomitant field terms, acoustic noise, or gradient heating limits is required — these are listed as future work not yet included
- Off-the-shelf vendor waveforms suffice and no arbitrary motion compensation, eddy-current nulling, or PNS constraint is needed

## Inputs

- Hardware limits: Gmax (T/m), Smax (T/m/s), gradient raster time dt, waveform size N
- Gradient moment constraints (M0, M1, M2) as moments_params arrays with 7 values per constraint
- Eddy-current nulling parameters (eddy_params, 4 values per constraint, tolerance ~1.0e-4, time constants e.g. 40/80 ms)
- Optional PNS constraint threshold (Schulte et al. single-exponential model) and target b-value (e.g. 1000 mm2/s)

## Outputs

- Time-optimal gradient waveform (G_out, N_out) per logical/physical axis with achieved duration T = N*dt
- Compute-time benchmarks (e.g. compute_times_diffusion.csv, outer_search_timings.csv)
- Verification of moment/b-value targets (e.g. vendor_waveform_comparison.csv with moments, b-value, duration)

## How to apply

Build the GrOpt C optimization kernels (src/optimize_kernel.c) and the Cython Python wrapper with 'python setup.py build_ext --inplace' from the python/ directory. Formulate the design as constraints: hardware Gmax and Smax, waveform size N and raster time dt, gradient moment constraints supplied as moments_params arrays (7 values per constraint for M0/M1/M2), and optionally eddy-current nulling (eddy_params, 4 values per constraint, tolerance ~1.0e-4, e.g. λ = 80 ms or λ = 40, 80 ms) or a Schulte et al. PNS constraint. Run minTE_diff()/run_kernel_diff(), which searches temporal durations T in a multi-threaded outer loop using higher dt for the initial solve and warm-starts the target-dt final optimization from those high-dt waveforms. Handle oblique slices via the rotationally invariant approach (derate Gmax and Smax by 3) or the rotationally variant approach (rotate constraints into physical coordinates and solve at full limits). If no feasible solution is found, the solver stops after 20,000 iterations; benchmark by averaging 100 optimization calls on a single-threaded desktop CPU (e.g. Intel Core i7-7700k, <10 Mb memory).

## Related tools

- **GrOpt** (Core ADMM-based optimizer solving the constrained waveform feasibility problem and outer time-optimal duration search) — https://github.com/cmr-group/gropt
- **C** (Language of the optimization kernels in src/optimize_kernel.c for native integration in pulse sequences) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C kernels to build the importable Python module via setup.py) — https://github.com/cmr-group/gropt
- **Python** (Wrapper for prototyping and visualizing gradient waveforms (Jupyter notebook demos in ./python/)) — https://github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace
```

## Evaluation signals

- Each returned waveform satisfies all constraints within the specified tolerance (bounds bi on ci(g) − ti) and stops after at most 20,000 iterations if infeasible
- Benchmarked compute times for acceptable waveforms fall within the reported 1-40 ms range (1-100 ms per README)
- Accelerated (multi-threaded, warm-started) duration searches are faster than a naive single-threaded scan over candidate durations T
- Residual gradient moments kept on the order of 10^-2 mT/m×ms (M0) or 10^-4 mT/m×ms² (M1) hold simulated ADC bias to ≤5%
- Waveforms generated on different vendor platforms (GE MR750/EPIC, Siemens Skyra/IDEA) meet their moment constraints and target b-value of 1000 mm2/s, confirming functional equivalence

## Limitations

- If no feasible solution exists or cannot be found, the optimization is stopped after 20,000 iterations
- With multiple constraints added, convexity of the solution space is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- There is no changelog or version information provided for the toolbox

## Evidence

- [methods] The GrOpt routine is based on the alternating direction method of multipliers (ADMM), with modifications made to handle any number of constraints: [evidence span withheld — anchor status: not_found]
