---
name: diffusion-encoding-waveform-design
description: Use when a diffusion MRI sequence (e.g., spin-echo EPI DWI) needs a shortest
  feasible gradient waveform for a target b-value while honoring Gmax/Smax hardware
  limits, gradient moment compensation, or eddy-current nulling.
when_to_use_negative:
- The goal is to optimize readout trajectories or slice-select gradients, which GrOpt
  has not been used for
- Long gradient waveforms such as multiecho turbo-spin echo sequences where the chosen
  N and dt may need adjustment
- A scenario requiring proof of convexity for a multi-constraint formulation, which
  is difficult to establish for this optimization
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: GrOpt
  role: ADMM-based optimization toolbox whose minTE_diff() routine searches for the
    shortest feasible diffusion-encoding waveform under moment and eddy-current constraints
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the optimization kernel (src/optimize_kernel.c) for maximum compatibility
    on scanner systems
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to invoke the C kernel, set Gmax/Smax/dt parameters, and write
    compute_times_diffusion.csv
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Bridges the C optimization routines to the Python module during the build_ext
    step
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:aa893d1358e9ebc34c15ca9adeeb1b0f4bf31a53c990abe8e54a98bf44397ea4
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
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# diffusion-encoding-waveform-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs time-optimal MRI diffusion-encoding gradient waveforms under gradient moment nulling (M0/M1/M2) and eddy-current nulling constraints using the GrOpt toolbox, targeting compute times of 1-40 ms suitable for on-the-fly protocol realization.

## When to use

Use when a diffusion MRI sequence (e.g., spin-echo EPI DWI) needs a shortest feasible gradient waveform for a target b-value while honoring Gmax/Smax hardware limits, gradient moment compensation, or eddy-current nulling.

## When NOT to use

- The goal is to optimize readout trajectories or slice-select gradients, which GrOpt has not been used for
- Long gradient waveforms such as multiecho turbo-spin echo sequences where the chosen N and dt may need adjustment
- A scenario requiring proof of convexity for a multi-constraint formulation, which is difficult to establish for this optimization

## Inputs

- Target diffusion b-value and temporal footprint
- Gradient hardware limits: Gmax (T/m) and Smax (T/m/s)
- Waveform size N and gradient raster time dt
- Gradient moment nulling constraints as moments_params arrays (7 values per M0/M1/M2 constraint)
- Eddy-current nulling constraints as eddy_params arrays (4 values per constraint) with eddy time constants (e.g., lambda = 80 ms; lambda = 40, 80 ms) and tolerance ~1.0e-4

## Outputs

- Time-optimal diffusion-encoding gradient waveform (gradient amplitude vs. time, N samples at raster dt)
- Convergence/feasibility status per configuration (20,000-iteration failure stop criterion)
- Per-configuration compute times and waveform parameters written to compute_times_diffusion.csv

## How to apply

Build the GrOpt C kernel (src/optimize_kernel.c) and the Cython wrapper via 'python setup.py build_ext --inplace' from the python/ directory. Set hardware limits (Gmax in T/m, Smax in T/m/s), waveform size N, and raster time dt, then run minTE_diff() which uses a multi-threaded outer search over waveform durations T, warm-starting the low-dt search with high-dt waveforms to solve quickly. Add gradient moment nulling by supplying moments_params arrays (7 values per constraint) for M0, M1, and/or M2, and eddy-current nulling via eddy_params (4 values per constraint, tolerance ~1.0e-4) for lambda = 80 ms or multiple time constants lambda = 40, 80 ms. The ADMM-based optimizer stops after 20,000 iterations if no feasible waveform is found, so treat non-convergence as infeasibility at that duration. Benchmark each configuration by averaging ~100 single-threaded calls and record compute times with waveform parameters; acceptable configurations should converge and fall in the reported 1-40 ms range.

## Related tools

- **GrOpt** (ADMM-based optimization toolbox whose minTE_diff() routine searches for the shortest feasible diffusion-encoding waveform under moment and eddy-current constraints) — github.com/cmr-group/gropt
- **C** (Language of the optimization kernel (src/optimize_kernel.c) for maximum compatibility on scanner systems) — github.com/cmr-group/gropt
- **Python** (Wrapper used to invoke the C kernel, set Gmax/Smax/dt parameters, and write compute_times_diffusion.csv) — github.com/cmr-group/gropt
- **Cython** (Bridges the C optimization routines to the Python module during the build_ext step) — github.com/cmr-group/gropt

## Examples

```
cd python/ && python setup.py build_ext --inplace
```

## Evaluation signals

- The optimizer converges to a feasible waveform for the target b-value rather than hitting the 20,000-iteration stop criterion
- Recorded compute time (average of 100 single-threaded calls) falls within the reported 1-40 ms range for acceptable gradient waveforms
- Eddy-current nulling configurations (lambda = 80 ms and lambda = 40, 80 ms) satisfy the 1.0e-4 tolerance on the eddy-current constraint
- Resulting waveform respects the specified Gmax and Smax hardware limits and the requested gradient moment nulling (M0/M1/M2)
- Per-configuration compute times and waveform parameters are written to compute_times_diffusion.csv

## Limitations

- Whether a waveform is truly time-optimal is difficult to prove; brute-force verification is computationally very expensive
- With multiple constraints on the gradient waveform design problem, convexity of the solution space becomes difficult to prove; randomized-start tests are not a complete proof
- The choice of gradient size N and raster time dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- There is no changelog or version information provided for the toolbox

## Evidence

- [readme] Added minTE_finder in src/optimize_kernel.c (minTE_diff function) to more efficiently fine the minimum TE: [evidence span withheld — anchor status: external_doc]
