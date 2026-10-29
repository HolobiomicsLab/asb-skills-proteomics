---
name: gradient-waveform-time-optimal-optimization
description: Use when a pulse sequence needs the shortest feasible gradient waveform
  under hardware limits (gmax, smax, raster time dt) plus encoding constraints such
  as diffusion b-value targets or gradient moment nulling orders.
when_to_use_negative:
- Optimizing readout trajectories or slice-select gradients — the article states the
  GrOpt toolbox has not been used to optimize these
- Problems needing constraint options not yet in the toolbox, such as concomitant
  field terms, acoustic noise, or gradient heating limits (listed as future work)
- Very long gradient waveforms (e.g., multiecho turbo-spin echo), where the default
  choice of gradient size N and raster time dt may need adjustment
tools:
- name: GrOpt
  role: Core toolbox solving the constrained gradient waveform feasibility/optimization
    problem via ADMM
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the optimization routines for native integration in pulse sequences,
    including minTE_diff in src/optimize_kernel.c
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into a Python module built with setup.py
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper for prototyping and visualizing gradient waveforms (Jupyter notebook
    demos)
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Wrapper compiled via make.m for prototyping and visualizing gradient waveforms
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:768cd6708d01d30229c3280142703b9368c30665294bed7e73eabef34171745d
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gradient-waveform-time-optimal-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design time-optimal MRI gradient waveforms (e.g., minimum-TE diffusion encoding with gradient moment nulling) by solving a constrained feasibility problem with the open-source GrOpt toolbox, whose optimization routines are written in C and exposed through Python (Cython) and Matlab wrappers.

## When to use

Use when a pulse sequence needs the shortest feasible gradient waveform under hardware limits (gmax, smax, raster time dt) plus encoding constraints such as diffusion b-value targets or gradient moment nulling orders.

## When NOT to use

- Optimizing readout trajectories or slice-select gradients — the article states the GrOpt toolbox has not been used to optimize these
- Problems needing constraint options not yet in the toolbox, such as concomitant field terms, acoustic noise, or gradient heating limits (listed as future work)
- Very long gradient waveforms (e.g., multiecho turbo-spin echo), where the default choice of gradient size N and raster time dt may need adjustment

## Inputs

- Gradient constraint set: gmax, smax, raster time dt, gradient moment nulling orders (e.g., M0/M1 nulling for diffusion encoding)
- Diffusion encoding target b-value (e.g., b = 1000 mm2/s) for the minTE_diff search
- Number of gradient waveform points N (waveform size for the sweep)
- Pulse-sequence context defining features and encodings for the acquisition

## Outputs

- Time-optimal gradient waveforms for each constraint configuration
- Minimum waveform duration / minimum TE per waveform size N, tabulated (e.g., min_T_vs_N.csv)
- Feasibility verdict per temporal duration T (or infeasible after 20,000 iterations)

## How to apply

Build the GrOpt C optimization routines and compile the Cython-wrapped Python module by running 'python setup.py build_ext --inplace' in the python/ directory. Define the constraint set for the acquisition (e.g., gmax, smax, dt, gradient moment nulling configuration, diffusion b-value maximization) and invoke the minTE_diff function in src/optimize_kernel.c, which lets the b-value search determine the shortest feasible waveform. Run the outer time-optimal search over temporal durations T in a multi-threaded fashion, solving the initial search with a higher dt for speed and warm-starting the target-dt final optimization with those high-dt waveforms. Apply the solver stopping rule: if no feasible solution exists or cannot be found, the process is stopped after 20,000 iterations. Record the minimum waveform duration T (= dt*N) per waveform size N (e.g., the sweep showing N=64 TEs within 1 ms and N=128 within 0.5 ms of the shortest TE) and export it, e.g., as min_T_vs_N.csv.

## Related tools

- **GrOpt** (Core toolbox solving the constrained gradient waveform feasibility/optimization problem via ADMM) — https://github.com/cmr-group/gropt
- **C** (Language of the optimization routines for native integration in pulse sequences, including minTE_diff in src/optimize_kernel.c) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into a Python module built with setup.py) — https://github.com/cmr-group/gropt
- **Python** (Wrapper for prototyping and visualizing gradient waveforms (Jupyter notebook demos)) — https://github.com/cmr-group/gropt
- **Matlab** (Wrapper compiled via make.m for prototyping and visualizing gradient waveforms) — https://github.com/cmr-group/gropt

## Examples

```
cd python/ && python setup.py build_ext --inplace
```

## Evaluation signals

- For N = 64 the minimum TEs are within 1 ms of the shortest TE, and for N = 128 within 0.5 ms — a convergence check on the N sweep
- Acceptable gradient waveforms are generated in 1-40 ms compute time, bounding solver runtime
- Infeasible configurations terminate after exactly 20,000 iterations rather than running indefinitely
- Waveforms generated and run on different scanners (e.g., GE EPIC and Siemens) are functionally equivalent and images comparable
- Output table (min_T_vs_N.csv) contains a monotonic decrease of minimum duration T = dt*N toward the time-optimal limit as N increases

## Limitations

- With multiple constraints, convexity of the solution space is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- The toolbox has not been used to optimize readout trajectories or slice-select gradients, and constraint options like concomitant field terms, acoustic noise, and gradient heating limits are future work

## Evidence

- [methods] ADMM solver basis: [evidence span withheld — anchor status: not_found]
- [readme] Build instructions: [evidence span withheld — anchor status: external_doc]
- [readme] minTE_diff function: [evidence span withheld — anchor status: external_doc]
