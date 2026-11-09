---
name: multithreaded-duration-search-benchmarking
description: Use when a constrained MRI gradient waveform optimization must find the
  shortest feasible total duration T quickly enough for on-the-fly scanner use, and
  a naive sequential scan over durations is too slow.
when_to_use_negative:
- The target is optimizing readout trajectories or slice-select gradients, which the
  GrOpt toolbox has not been used to optimize
- A single fixed-duration feasibility solve is all that is needed — the outer duration
  search and its warm-start machinery add overhead without benefit
- The waveform is so long (e.g. multiecho turbo-spin echo) that the default choice
  of N and dt may need adjustment before any timing benchmark is meaningful
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3306
- http://edamontology.org/topic_3452
tools:
- name: GrOpt
  role: Core optimization toolbox whose outer-loop duration search (multi-threaded
    T sweep, coarse-to-fine dt, warm starting) is benchmarked
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the optimization kernels (src/optimize_kernel.c) including the
    OpenMP TE finder and minTE_diff
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines for the Python benchmarking harness, built
    via setup.py build_ext --inplace
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper environment used to invoke the kernels and time 100 individual optimization
    calls
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:b90e2a49d056df91fbeb1a83f7b0fe2c0c8e81167984b1e63cd35fe3072b4517
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
  edam_topics:
  - http://edamontology.org/topic_3306
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multithreaded-duration-search-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An accelerated outer-loop strategy from GrOpt for finding the time-optimal gradient waveform duration T: candidate durations are searched in a multi-threaded fashion, each first solved at a coarser (higher) dt and then warm-started at the target dt, enabling on-the-fly waveform generation in 1-40 ms.

## When to use

Use when a constrained MRI gradient waveform optimization must find the shortest feasible total duration T quickly enough for on-the-fly scanner use, and a naive sequential scan over durations is too slow.

## When NOT to use

- The target is optimizing readout trajectories or slice-select gradients, which the GrOpt toolbox has not been used to optimize
- A single fixed-duration feasibility solve is all that is needed — the outer duration search and its warm-start machinery add overhead without benefit
- The waveform is so long (e.g. multiecho turbo-spin echo) that the default choice of N and dt may need adjustment before any timing benchmark is meaningful

## Inputs

- Inner GrOpt problem specification: gmax, smax, fixed N and dt, moment constraints in the 7-value moments_params format
- Candidate set of total waveform durations T = dt*N
- Compiled GrOpt C kernels and Cython Python wrapper (src/optimize_kernel.c)

## Outputs

- Time-optimal gradient waveform g(t) at the target dt satisfying all constraints
- Per-duration, per-strategy execution timings (e.g. written to outer_search_timings.csv), averaged over 100 calls
- Feasibility/iteration status per candidate duration (cap of 20,000 iterations when infeasible)

## How to apply

Formulate the inner problem with run_kernel_diff()/minTE_diff() in src/optimize_kernel.c, fixing N and dt and specifying gmax, smax, and moment constraints (7-value moments_params format). Build the C kernels plus the Cython wrapper via 'python setup.py build_ext --inplace'. Replace a naive single-threaded baseline that sequentially scans candidate durations T = dt*N with the accelerated scheme: sweep durations T in a multi-threaded fashion (the OpenMP TE finder calls multiple GrOpt evals simultaneously to find the shortest feasible T), solve each candidate first at a higher dt for speed, then warm start the final target-dt optimization from those high-dt waveforms. Benchmark by averaging the duration of 100 individual optimization calls on a single-threaded desktop CPU (e.g. Intel Core i7-7700k class), recording timings per duration and strategy. Confirm returned waveforms satisfy all constraints (bounds bi on ci(g) - ti) within tolerance and that infeasible solves stop after 20,000 iterations.

## Related tools

- **GrOpt** (Core optimization toolbox whose outer-loop duration search (multi-threaded T sweep, coarse-to-fine dt, warm starting) is benchmarked) — https://github.com/cmr-group/gropt
- **C** (Language of the optimization kernels (src/optimize_kernel.c) including the OpenMP TE finder and minTE_diff) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines for the Python benchmarking harness, built via setup.py build_ext --inplace) — https://github.com/cmr-group/gropt
- **Python** (Wrapper environment used to invoke the kernels and time 100 individual optimization calls)

## Examples

```
cd python && python setup.py build_ext --inplace  # then call the OpenMP TE finder (test_TE_finder / minTE_diff) in src/optimize_kernel.c to find the shortest feasible T
```

## Evaluation signals

- Every returned waveform satisfies all constraints within the specified tolerance (bounds bi on ci(g) - ti)
- Infeasible solves terminate at the 20,000-iteration cap rather than running indefinitely
- Accelerated-search timings fall within the paper's reported 1-40 ms generation range
- Multi-threaded warm-started search is faster than the naive single-threaded sequential scan over durations T = dt*N
- Timing averages are computed over 100 individual calls on a single-threaded desktop CPU (e.g. Intel Core i7-7700k class) and recorded per duration and strategy

## Limitations

- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- With multiple constraints, convexity of the solution space is difficult to prove, and randomized-start tests are not a complete proof
- The choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences

## Evidence

- [readme] OpenMP version of TE finder is better implemented. See demo in the `test_TE_finder` function of src/optimize_kernel.c, which will call multiple GrOpt evals simultaneously to find the shortest feasible T.: [evidence span withheld — anchor status: external_doc]
