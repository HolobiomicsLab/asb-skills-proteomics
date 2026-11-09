---
name: gradient-moment-nulling-constraint-design
description: Use when generating time-optimal diffusion gradient waveforms that must
  suppress motion-induced phase errors (gradient moment compensation up to M2) or
  eddy-current effects (λ = 40, 80 ms), and when benchmarking on-the-fly generation
  speed vs. gradient raster time dt.
when_to_use_negative:
- Optimizing readout trajectories or slice-select gradients — the GrOpt toolbox has
  not been used for these applications
- When constraints beyond those implemented (e.g., concomitant field terms, acoustic
  noise, gradient heating limits) are required — these are future work and not yet
  included
- When exhaustive proof of true time-optimality is required — brute-force verification
  is computationally very expensive and convexity with multiple constraints is difficult
  to prove
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3452
tools:
- name: GrOpt
  role: ADMM-based gradient waveform optimizer providing moment nulling, eddy-current
    nulling, and minTE_diff time-optimal search
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the core optimization routines (src/optimize_kernel.c) benchmarked
    single-threaded with <10 Mb memory
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into a Python module built via setup.py
    build_ext --inplace
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to invoke the compiled GrOpt module for waveform design and benchmarking
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:d6d2f7b86e983964f6c9a48e9acc0f51e23ab19c354bfa68bd328261d63b6bb2
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# gradient-moment-nulling-constraint-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design time-optimal MRI diffusion-encoding gradient waveforms under gradient moment nulling (M0/M1/M2) and eddy-current nulling constraints using the GrOpt toolbox (C kernel with Cython/Python wrapper), benchmarking compute time as a function of gradient raster time.

## When to use

Use when generating time-optimal diffusion gradient waveforms that must suppress motion-induced phase errors (gradient moment compensation up to M2) or eddy-current effects (λ = 40, 80 ms), and when benchmarking on-the-fly generation speed vs. gradient raster time dt.

## When NOT to use

- Optimizing readout trajectories or slice-select gradients — the GrOpt toolbox has not been used for these applications
- When constraints beyond those implemented (e.g., concomitant field terms, acoustic noise, gradient heating limits) are required — these are future work and not yet included
- When exhaustive proof of true time-optimality is required — brute-force verification is computationally very expensive and convexity with multiple constraints is difficult to prove

## Inputs

- Hardware limits (Gmax in T/m, Smax in T/m/s)
- Waveform size N and gradient raster time dt
- Target diffusion b-value
- moments_params arrays (7 values per gradient moment constraint, M0/M1/M2)
- eddy_params arrays (4 values per eddy-current constraint; time constants λ = 40 ms, 80 ms)

## Outputs

- Time-optimal diffusion-encoding gradient waveform (per configuration)
- compute_times_diffusion.csv with per-configuration compute times and waveform parameters
- Convergence/feasibility status (20,000-iteration failure stop criterion)

## How to apply

Build the GrOpt C optimization kernel (src/optimize_kernel.c) and compile the Cython wrapper via 'python setup.py build_ext --inplace' from the python/ directory. Set hardware limits (Gmax in T/m, Smax in T/m/s), waveform size N, and raster time dt, then run minTE_diff() which performs an outer multi-threaded duration search, warm-starting from higher-dt solutions to find the shortest feasible T for a target b-value. Add gradient moment nulling via moments_params arrays (7 values per constraint for M0/M1/M2) and eddy-current nulling via eddy_params (4 values per constraint, tolerance ≈1.0e-4) for no nulling, single time constant λ = 80 ms, or multiple time constants λ = 40, 80 ms. Benchmark each configuration by averaging 100 single-threaded calls to the C routines (minimal memory, <10 Mb) and record per-configuration compute times and waveform parameters to compute_times_diffusion.csv. The GrOpt routine is ADMM-based; if no feasible solution is found within 20,000 iterations, treat the configuration as failed. Acceptable waveforms should converge with compute times in the 1-40 ms range, with a trade-off between computation time and gradient raster time.

## Related tools

- **GrOpt** (ADMM-based gradient waveform optimizer providing moment nulling, eddy-current nulling, and minTE_diff time-optimal search) — github.com/cmr-group/gropt
- **C** (Language of the core optimization routines (src/optimize_kernel.c) benchmarked single-threaded with <10 Mb memory) — github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into a Python module built via setup.py build_ext --inplace) — github.com/cmr-group/gropt
- **Python** (Wrapper used to invoke the compiled GrOpt module for waveform design and benchmarking) — github.com/cmr-group/gropt

## Examples

```
python setup.py build_ext --inplace  # from python/ directory, then call minTE_diff() with Gmax, Smax, N, dt, b-value, moments_params, and eddy_params
```

## Evaluation signals

- Optimization converges to a feasible waveform (failure stop criterion is 20,000 iterations without feasibility)
- Compute time falls within the reported 1-40 ms range for acceptable gradient waveforms
- Benchmark averaging uses 100 individual single-threaded calls with minimal memory (<10 Mb)
- Waveforms satisfy all supplied constraints within specified tolerances (eddy-current tolerance ≈1.0e-4; moments enforced to the requested order M0/M1/M2)
- Compute time increases as gradient raster time dt decreases, consistent with the reported raster-time trade-off

## Limitations

- With multiple constraints, convexity of the solution space is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force approaches are computationally very expensive
- Choice of waveform size N and raster time dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Compute time depends on gradient raster time — a trade-off exists between computation time and gradient raster time

## Evidence

- [methods] ADMM solver with iteration cap: [evidence span withheld — anchor status: not_found]
- [readme] minTE_diff addition: [evidence span withheld — anchor status: external_doc]
