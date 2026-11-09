---
name: diffusion-encoding-moment-nulling-configuration
description: Use when designing time-optimal diffusion-encoding gradient waveforms
  (e.g., b = 1000 mm2/s) with moment nulling or eddy-current nulling constraints and
  choosing N/dt so minimum TE is within ~1 ms of the shortest feasible TE.
when_to_use_negative:
- The target is optimizing readout trajectories rather than diffusion encoding waveforms
  — the article notes GrOpt 'has not been used to optimize readout trajectories'
- The goal is benchmarking compute-time scaling exponents rather than producing a
  feasible time-optimal diffusion waveform
- Waveforms are much longer than typical (e.g., multiecho turbo-spin echo), where
  the article warns the choice of N and dt may need adjustment
tools:
- name: GrOpt
  role: C optimization toolbox providing minTE_diff and moment/eddy-current nulling
    constraints used to compute the minimum-TE diffusion waveform
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the optimization routines in src/ compiled for the solver
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper (Cython-built module) through which minTE_diff is called for the N
    sweep
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Build layer used with 'python setup.py build_ext --inplace' to expose the
    C routines to Python
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:8c58397804c22deefb71dd0b0a5ef3260354a1f5f27147e1e5744b2d643ee164
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Diffusion encoding moment nulling configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure GrOpt diffusion-encoding gradient waveform design with gradient moment nulling (M0/M1/M2) and eddy-current nulling constraints, sweeping waveform point count N to obtain minimum echo times approaching the time-optimal limit.

## When to use

Use when designing time-optimal diffusion-encoding gradient waveforms (e.g., b = 1000 mm2/s) with moment nulling or eddy-current nulling constraints and choosing N/dt so minimum TE is within ~1 ms of the shortest feasible TE.

## When NOT to use

- The target is optimizing readout trajectories rather than diffusion encoding waveforms — the article notes GrOpt 'has not been used to optimize readout trajectories'
- The goal is benchmarking compute-time scaling exponents rather than producing a feasible time-optimal diffusion waveform
- Waveforms are much longer than typical (e.g., multiecho turbo-spin echo), where the article warns the choice of N and dt may need adjustment

## Inputs

- Diffusion encoding constraint set (gmax, smax, raster time dt, target b-value such as 1000 mm2/s)
- Gradient moment nulling configuration (M0, M1, M2 or partial nulling orders)
- Optional eddy-current nulling time constants
- Waveform point count N (e.g., 64 and 128) and raster time dt

## Outputs

- Minimum gradient waveform duration T = dt*N (minimum TE) per (moment-nulling configuration, N) combination
- Time-optimal diffusion gradient waveform (e.g., min_T_vs_N.csv summary table)

## How to apply

Build the GrOpt C routines and compile the Cython-wrapped Python module with 'python setup.py build_ext --inplace' from the python/ directory. For each waveform size N, call minTE_diff() in src/optimize_kernel.c with constraints on gmax, smax, raster time dt, and the chosen gradient moment nulling order (full or partial nulling of any moment M_n), optionally adding eddy-current nulling for one or more eddy current time constants. Run the outer time-optimal search over temporal durations T, first solving with higher dt for speed and warm-starting the target-dt optimization with those waveforms. Apply the solver stopping rule of terminating after 20,000 iterations if no feasible solution is found, then record the minimum duration T = dt*N per configuration and verify that N > 64 puts the minimum TE within 1 ms of the shortest TE (within 0.5 ms at N = 128).

## Related tools

- **GrOpt** (C optimization toolbox providing minTE_diff and moment/eddy-current nulling constraints used to compute the minimum-TE diffusion waveform) — github.com/cmr-group/gropt
- **C** (Language of the optimization routines in src/ compiled for the solver) — github.com/cmr-group/gropt
- **Python** (Wrapper (Cython-built module) through which minTE_diff is called for the N sweep) — github.com/cmr-group/gropt
- **Cython** (Build layer used with 'python setup.py build_ext --inplace' to expose the C routines to Python) — github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace
```

## Evaluation signals

- For N = 64 the minimum TE is within 1 ms of the shortest feasible TE; for N = 128 it is within 0.5 ms
- Solver terminates after 20,000 iterations when no feasible solution exists, so infeasible configurations must be flagged, not silently returned
- Fitted waveform duration T must equal dt*N for the chosen raster time
- Resulting waveforms respect gmax, smax, moment-nulling, and eddy-current-nulling constraints within the specified tolerance
- Benchmarked waveform generation times stay in the 1-40 ms range reported for acceptable waveforms

## Limitations

- Convexity of the solution space is difficult to prove when multiple constraints are combined, so solutions may be local rather than globally optimal
- Whether a waveform is truly time-optimal is difficult to verify; brute-force checks are computationally very expensive
- The choice of gradient size (N) and raster time (dt) may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- No changelog or version information is provided for the toolbox

## Evidence

- [readme] minTE_diff added to optimize_kernel.c: [evidence span withheld — anchor status: external_doc]
