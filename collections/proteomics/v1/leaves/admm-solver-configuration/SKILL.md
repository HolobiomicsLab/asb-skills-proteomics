---
name: admm-solver-configuration
description: Use when you need to solve a constrained MRI gradient waveform design
  problem — e.g. gmax, smax, gradient moment nulling, eddy current or PNS constraints
  — via GrOpt's ADMM solver and need correct stopping, tolerance, and warm-start configuration.
when_to_use_negative:
- The task is optimizing readout trajectories or slice-select gradients — the article
  states the GrOpt toolbox has not been used to optimize readout trajectories or slice-select
  gradients
- You need constraints not yet in GrOpt, such as concomitant field terms, acoustic
  noise, or gradient heating limits — these are listed as future work
- Very long waveforms (e.g. multiecho turbo-spin echo), where the default choice of
  N and dt may need adjustment
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3444
tools:
- name: GrOpt
  role: Core toolbox whose ADMM-based optimization routines perform the constrained
    gradient waveform feasibility solve
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the optimization routines (src/optimize_kernel.c) written for
    maximum compatibility on multiple scanner systems
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into a Python module built via setup.py
    build_ext --inplace
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used for prototyping and visualizing gradient waveforms (Jupyter notebook
    demos in ./python/)
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Wrapper compiled via make.m with mex for prototyping and visualization
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:b7e39699f2cf9bbe3fc8a34096f61a85d2d5abc7cd50aeca68a22f2f0a0b1581
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
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# admm-solver-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run GrOpt's ADMM-based feasibility solver (implemented in C, wrapped with Cython for Python) to find gradient waveforms satisfying hardware and moment constraints, with the 20,000-iteration stopping rule and tolerance-based feasibility checks on ci(g) - ti bounds.

## When to use

Use when you need to solve a constrained MRI gradient waveform design problem — e.g. gmax, smax, gradient moment nulling, eddy current or PNS constraints — via GrOpt's ADMM solver and need correct stopping, tolerance, and warm-start configuration.

## When NOT to use

- The task is optimizing readout trajectories or slice-select gradients — the article states the GrOpt toolbox has not been used to optimize readout trajectories or slice-select gradients
- You need constraints not yet in GrOpt, such as concomitant field terms, acoustic noise, or gradient heating limits — these are listed as future work
- Very long waveforms (e.g. multiecho turbo-spin echo), where the default choice of N and dt may need adjustment

## Inputs

- Set of input constraints defining the pulse sequence (gmax, smax, raster time dt, gradient moment nulling via 7-value moments_params, eddy current and PNS constraints)
- Fixed waveform length N and duration T = dt*N
- Compiled GrOpt C optimization kernels (src/optimize_kernel.c) with Python or Matlab wrapper

## Outputs

- Feasible/time-optimal gradient waveform g(t) satisfying all constraints within tolerance
- Achieved diffusion b-value (for diffusion encoding problems solved via b-value maximization)
- Solver status indicating feasibility or termination at 20,000 iterations without a feasible solution

## How to apply

Compile the GrOpt C kernels in src/optimize_kernel.c and build the Cython wrapper with 'python setup.py build_ext --inplace' from the python/ directory (or compile Matlab mex via make.m). Formulate the inner feasibility problem with run_kernel_diff()/minTE_diff(), specifying gmax, smax, fixed N and raster time dt, and moment constraints in the 7-value moments_params format; GrOpt's routine is based on the alternating direction method of multipliers (ADMM) with modifications to handle any number of constraints, applied modularly. If no feasible solution is found, terminate after 20,000 iterations rather than iterating indefinitely. Verify each returned waveform satisfies all constraints within the specified tolerance on bounds bi applied to ci(g) - ti. For speed, warm-start the target-dt final optimization from higher-dt solutions found during the multi-threaded outer duration (T) search, which yields acceptable waveforms in 1-40 ms on a desktop-class CPU.

## Related tools

- **GrOpt** (Core toolbox whose ADMM-based optimization routines perform the constrained gradient waveform feasibility solve) — https://github.com/cmr-group/gropt
- **C** (Language of the optimization routines (src/optimize_kernel.c) written for maximum compatibility on multiple scanner systems) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into a Python module built via setup.py build_ext --inplace) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used for prototyping and visualizing gradient waveforms (Jupyter notebook demos in ./python/)) — https://github.com/cmr-group/gropt
- **Matlab** (Wrapper compiled via make.m with mex for prototyping and visualization) — https://github.com/cmr-group/gropt

## Examples

```
python setup.py build_ext --inplace
```

## Evaluation signals

- Each returned waveform satisfies all constraints within the specified tolerance (bounds bi on ci(g) - ti)
- Solver stops after at most 20,000 iterations when no feasible solution exists
- Generation times fall within the reported 1-40 ms range on a single-threaded desktop CPU (e.g. Intel Core i7-7700k class)
- Warm-started target-dt solve is faster than a naive single-threaded baseline scan over candidate durations T
- Waveforms generated and run on different scanners are functionally equivalent with comparable images

## Limitations

- With multiple constraints, convexity of the solution space becomes difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Residual gradient moments from non-zero convergence criteria can affect intravoxel phase dispersion and ADC (assessed via numerical simulation)
- No changelog or version information is provided for the toolbox

## Evidence

- [methods] The GrOpt routine is based on the alternating direction method of multipliers (ADMM), with modifications made to handle any number of constraints: [evidence span withheld — anchor status: not_found]
- [methods] If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations: "If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations"
- [methods] input constraints are given, and the optimization routine seeks a gradient waveform subject to the constraints within a specified tolerance: "input constraints are given, and the optimization routine seeks a gradient waveform subject to the constraints within a specified tolerance"
- [abstract] it was observed that acceptable gradient waveforms could be generated in 1-40 ms.: "it was observed that acceptable gradient waveforms could be generated in 1-40 ms."
- [methods] The optimization routines have been wrapped with Cython, they can be built simply running in the python/ directory: "The optimization routines have been wrapped with Cython, they can be built simply running in the python/ directory"
- [readme] OpenMP version of TE finder is better implemented. See demo in the `test_TE_finder` function of src/optimize_kernel.c: [evidence span withheld — anchor status: external_doc]
