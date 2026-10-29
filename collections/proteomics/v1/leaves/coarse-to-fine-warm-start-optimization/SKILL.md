---
name: coarse-to-fine-warm-start-optimization
description: Use when a time-optimal MRI gradient waveform duration T must be found
  on-the-fly (within 1-40 ms) and a naive single-threaded sequential scan over durations
  T = dt*N is too slow.
when_to_use_negative:
- The task is designing a waveform at a known, fixed duration T — only the inner feasibility/objective
  optimization is needed, not the outer duration search
- Readout trajectories or slice-select gradients must be optimized — the article states
  GrOpt has not been used for these
- A brute-force proof of true time-optimality is required — the article notes brute-force
  verification is computationally very expensive and convexity with multiple constraints
  is difficult to prove
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3306
- http://edamontology.org/topic_3399
tools:
- name: GrOpt
  role: Provides the ADMM-based inner feasibility solver and the multi-threaded OpenMP
    TE finder / minTE_diff function used to sweep durations T
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the optimization kernels in src/optimize_kernel.c compiled for
    maximum compatibility on scanner systems
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C routines into a Python module built via 'python setup.py build_ext
    --inplace'
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used for prototyping, benchmarking the naive vs. accelerated outer
    searches, and writing outer_search_timings.csv
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:3efab15d4b46f48b39c702cf17f03f86a40e855ef4c0d20022c243c93ebc4f09
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3306
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# coarse-to-fine-warm-start-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Accelerates GrOpt's outer time-optimal search for gradient waveform duration T by sweeping candidate durations T multi-threaded, first solving each with a coarser (higher) dt, then warm-starting the finer dt-resolution optimization from the coarse waveforms.

## When to use

Use when a time-optimal MRI gradient waveform duration T must be found on-the-fly (within 1-40 ms) and a naive single-threaded sequential scan over durations T = dt*N is too slow.

## When NOT to use

- The task is designing a waveform at a known, fixed duration T — only the inner feasibility/objective optimization is needed, not the outer duration search
- Readout trajectories or slice-select gradients must be optimized — the article states GrOpt has not been used for these
- A brute-force proof of true time-optimality is required — the article notes brute-force verification is computationally very expensive and convexity with multiple constraints is difficult to prove

## Inputs

- Inner-loop problem definition: gmax, smax, fixed N and dt, and moment constraints in the 7-value moments_params format
- Candidate set of total durations T = dt*N to sweep
- Compiled GrOpt C kernels (src/optimize_kernel.c) with OpenMP-enabled TE finder
- Timing harness averaging 100 individual optimization calls on a desktop CPU

## Outputs

- Time-optimal gradient waveform g(t) at the target dt satisfying all constraint bounds bi on ci(g) - ti within tolerance
- Per-duration and per-strategy execution timings written to outer_search_timings.csv
- Feasibility flags; infeasible problems reported after at most 20,000 iterations

## How to apply

Compile the GrOpt C kernels (src/optimize_kernel.c) and build the Cython wrapper with 'python setup.py build_ext --inplace' from the python/ directory. Formulate the inner constrained problem with run_kernel_diff()/minTE_diff(), fixing N and dt, and specifying gmax, smax, and moment constraints in the 7-value moments_params format. Implement a naive baseline that sequentially scans candidate durations T = dt*N at fixed dt, then implement the accelerated outer search: sweep temporal durations T in a multi-threaded fashion (OpenMP TE finder calls multiple GrOpt evals simultaneously to find the shortest feasible T), performing the initial search of each T with a higher dt to solve quickly, and using those high-dt waveforms to warm start the target-dt final optimization. Benchmark by averaging the duration of 100 individual optimization calls on a single-threaded desktop CPU (e.g. Intel Core i7-7700k class). The rationale is that coarse-dt solutions are cheap to compute but provide good initializations, so the fine-dt ADMM solver converges in far fewer of its (max 20,000) iterations.

## Related tools

- **GrOpt** (Provides the ADMM-based inner feasibility solver and the multi-threaded OpenMP TE finder / minTE_diff function used to sweep durations T) — github.com/cmr-group/gropt
- **C** (Language of the optimization kernels in src/optimize_kernel.c compiled for maximum compatibility on scanner systems) — github.com/cmr-group/gropt
- **Cython** (Wraps the C routines into a Python module built via 'python setup.py build_ext --inplace') — github.com/cmr-group/gropt
- **Python** (Wrapper used for prototyping, benchmarking the naive vs. accelerated outer searches, and writing outer_search_timings.csv) — github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace  # then call run_kernel_diff()/minTE_diff() per candidate T, warm-starting the target-dt solve from the high-dt waveform
```

## Evaluation signals

- Each returned waveform satisfies all constraints (bounds bi on ci(g) - ti) within the specified tolerance
- Infeasible problems terminate after at most 20,000 iterations rather than hanging
- Accelerated-search timings fall within the paper's reported 1-40 ms waveform generation range
- Accelerated multi-threaded coarse-to-fine search is faster than the naive single-threaded sequential scan over T = dt*N
- Timings are averaged over 100 calls per duration/strategy and written to outer_search_timings.csv

## Limitations

- Whether a waveform is truly time-optimal is difficult to answer; brute-force approaches are computationally very expensive
- Convexity of the solution space with multiple constraints is difficult to prove, so the coarse-dt warm start could in principle bias the fine-dt solution
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Reported 1-40 ms timings were measured on a single-threaded desktop CPU (Intel Core i7-7700k class); scanner hardware may differ

## Evidence

- [methods] searching different temporal durations (T) in a multi-threaded fashion and performing the initial search of T with higher dt to solve quickly: [evidence span withheld — anchor status: not_found]
- [methods] If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations: "If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations"
- [abstract] it was observed that acceptable gradient waveforms could be generated in 1-40 ms.: "it was observed that acceptable gradient waveforms could be generated in 1-40 ms."
- [readme] OpenMP version of TE finder is better implemented. See demo in the `test_TE_finder` function of src/optimize_kernel.c, which will call multiple GrOpt evals simultaneously to find the shortest feasible T.: [evidence span withheld — anchor status: external_doc]
- [methods] The optimization routines have been wrapped with Cython, they can be built simply running in the python/ directory: "The optimization routines have been wrapped with Cython, they can be built simply running in the python/ directory"
