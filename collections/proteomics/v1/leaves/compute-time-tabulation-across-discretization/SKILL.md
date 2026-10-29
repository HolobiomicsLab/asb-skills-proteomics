---
name: compute-time-tabulation-across-discretization
description: Use when you need to quantify how a chosen gradient waveform discretization
  N affects the minimum TE (or compute feasibility) of an optimized diffusion waveform,
  e.g. to justify a raster-time/point-count choice before production waveform design.
when_to_use_negative:
- The goal is optimizing readout trajectories or slice-select gradients — the article
  states GrOpt has not been used for these
- The waveform is much longer than a typical diffusion encoding (e.g. multiecho turbo-spin
  echo), where the article warns the choice of N and dt may need adjustment
- A single fixed-N waveform design is already established and no discretization sensitivity
  analysis is needed
edam_operation: http://edamontology.org/operation_2945
tools:
- name: GrOpt
  role: Optimization toolbox providing the minTE_diff C function and time-optimal
    search used to find the shortest feasible waveform for each N
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the core optimization routines in src/ for maximum compatibility
    on scanner systems
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to invoke the compiled GrOpt module during the N sweep
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines for the Python module built with setup.py
    build_ext --inplace
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:ddd038b4a5c75d2366ad7d4c895c963e1ec4dcbf8d6dc3194e39f82e5aa51b23
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# compute-time-tabulation-across-discretization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweeps the number of gradient waveform points N in GrOpt and tabulates the minimum achievable echo time (minimum waveform duration T = dt*N) for diffusion encoding at a target b-value, quantifying how discretization converges toward the time-optimal limit.

## When to use

Use when you need to quantify how a chosen gradient waveform discretization N affects the minimum TE (or compute feasibility) of an optimized diffusion waveform, e.g. to justify a raster-time/point-count choice before production waveform design.

## When NOT to use

- The goal is optimizing readout trajectories or slice-select gradients — the article states GrOpt has not been used for these
- The waveform is much longer than a typical diffusion encoding (e.g. multiecho turbo-spin echo), where the article warns the choice of N and dt may need adjustment
- A single fixed-N waveform design is already established and no discretization sensitivity analysis is needed

## Inputs

- Gradient hardware limits: maximum gradient amplitude gmax and maximum slew smax
- Gradient raster time dt and waveform point count N to sweep (e.g. N = 64, 128)
- Diffusion encoding constraints: target b-value (e.g. b = 1000 mm2/s) and gradient moment nulling configuration

## Outputs

- Table (min_T_vs_N.csv) of minimum waveform duration T (= dt*N) per N
- Convergence assessment of minimum TE vs the time-optimal limit (e.g. within 1 ms for N > 64, within 0.5 ms for N = 128)

## How to apply

Build the GrOpt C optimization routines into a Python module by running 'python setup.py build_ext --inplace' in the python/ directory (Cython-wrapped, tested with Anaconda/Python 3.7). For each waveform size N in the sweep, call the minTE_diff() function (added in src/optimize_kernel.c) with diffusion constraints: gmax, smax, raster time dt, and the chosen gradient moment nulling configuration, letting the b-value search determine the shortest feasible waveform. Run the outer time-optimal search over temporal durations T — performing the initial search of T with higher dt to solve quickly, then warm-starting the target-dt final optimization from those high-dt waveforms. Apply the solver stopping rule of 20,000 iterations if no feasible solution is found. Record the minimum waveform duration T = dt*N for each N into a table (min_T_vs_N.csv), and verify that for N > 64 the resulting TEs are within 1 ms of the shortest TE.

## Related tools

- **GrOpt** (Optimization toolbox providing the minTE_diff C function and time-optimal search used to find the shortest feasible waveform for each N) — github.com/cmr-group/gropt
- **C** (Language of the core optimization routines in src/ for maximum compatibility on scanner systems) — github.com/cmr-group/gropt
- **Python** (Wrapper used to invoke the compiled GrOpt module during the N sweep) — github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines for the Python module built with setup.py build_ext --inplace) — github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace
```

## Evaluation signals

- For N = 64, the minimum TEs should fall within 1 ms of the shortest TE; for N = 128, within 0.5 ms
- Each recorded minimum duration must equal dt*N exactly (consistency check on the tabulated T values)
- Infeasible configurations must terminate after 20,000 iterations rather than hanging
- Warm-started target-dt optimization should converge to the same T as the fast high-dt initial search, within tolerance
- Compute times for feasible solves should fall in the milliseconds range (acceptable waveforms generated in 1-40 ms)

## Limitations

- If no feasible solution exists or cannot be found, the optimization is simply stopped after 20,000 iterations — infeasibility is not distinguishable from slow convergence
- The choice of gradient size (N) and raster time (dt) may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Whether a waveform is truly time-optimal is difficult to prove; brute-force verification is computationally very expensive
- With multiple constraints, convexity of the solution space is difficult to prove, so a found minimum may not be globally optimal

## Evidence

- [readme] minTE_diff function in optimize_kernel.c: [evidence span withheld — anchor status: external_doc]
