---
name: eddy-current-nulling-constraint-configuration
description: Use when designing time-optimal MRI gradient waveforms (e.g., diffusion
  encoding in spin-echo EPI DWI) where eddy-current-induced image artifacts must be
  suppressed by nulling specific eddy-current time constants via GrOpt constraints.
when_to_use_negative:
- The target is optimizing readout trajectories or slice-select gradients — the article
  states GrOpt has not been used for those
- The design goal is gradient duty-cycle minimization or diffusion b-value maximization
  rather than eddy-current artifact suppression (different GrOpt constraints exist
  for those)
- Only peripheral nerve stimulation (PNS) limits matter — use the Schulte et al. /
  Hebrank et al. PNS constraints instead of the eddy-current constraint
tools:
- name: GrOpt
  role: Core optimization toolbox that implements the eddy-current nulling constraint
    and the time-optimal waveform search
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the optimization routines (src/optimize_kernel.c, minTE_diff)
    that the eddy-current constraints are applied within
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into a Python module so eddy_params and
    moments_params can be passed from Python
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to configure constraints and run the optimization workflows
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:16d46b3970562e7597e64e04ab831f3e6916f66131885c58403c76858663db1d
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
yes_rate: 0.6666666666666666
n_claims: 12
---

# eddy-current-nulling-constraint-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures the GrOpt eddy-current nulling constraint (eddy_params arrays with one entry per eddy-current time constant) so that time-optimal diffusion-encoding gradient waveforms suppress the effects of one or more eddy-current time constants while meeting hardware and moment constraints.

## When to use

Use when designing time-optimal MRI gradient waveforms (e.g., diffusion encoding in spin-echo EPI DWI) where eddy-current-induced image artifacts must be suppressed by nulling specific eddy-current time constants via GrOpt constraints.

## When NOT to use

- The target is optimizing readout trajectories or slice-select gradients — the article states GrOpt has not been used for those
- The design goal is gradient duty-cycle minimization or diffusion b-value maximization rather than eddy-current artifact suppression (different GrOpt constraints exist for those)
- Only peripheral nerve stimulation (PNS) limits matter — use the Schulte et al. / Hebrank et al. PNS constraints instead of the eddy-current constraint

## Inputs

- eddy_params array (4 values per eddy-current constraint: time constant λ such as 80 ms or λ = 40, 80 ms)
- Hardware limits: Gmax (T/m) and Smax (T/m/s), waveform size N, gradient raster time dt
- moments_params arrays (7 values per gradient moment constraint, for M0/M1/M2 nulling variants)
- Target b-value for diffusion encoding
- Convergence tolerance (around 1.0e-4) and iteration cap (20,000)

## Outputs

- Time-optimal gradient waveform with eddy-current nulling applied (per-axis gradient samples)
- Feasibility/convergence flag per configuration (20,000-iteration stop criterion)
- Per-configuration compute times (averaged over 100 calls) written to compute_times_diffusion.csv

## How to apply

Add the eddy-current constraint alongside the core optimization inputs (Gmax in T/m, Smax in T/m/s, waveform size N, raster time dt) and any gradient moment nulling constraints (moments_params with 7 values per M0/M1/M2 constraint). Specify eddy-current conditions as eddy_params with 4 values per constraint: no nulling, nulling of the single time constant λ = 80 ms, or nulling of multiple time constants λ = 40, 80 ms, with a convergence tolerance around 1.0e-4. Run the optimization through minTE_diff() in src/optimize_kernel.c (built in C, wrapped via Cython with 'python setup.py build_ext --inplace'), which uses the outer multi-threaded duration search with high-dt warm starting to find the shortest feasible waveform. If no feasible solution exists within 20,000 iterations, the optimization stops and the configuration should be flagged as failed rather than reported as time-optimal. Benchmark each nulling configuration by averaging 100 single-threaded, low-memory (<10 MB) optimization calls to verify the 1-40 ms acceptable compute-time regime.

## Related tools

- **GrOpt** (Core optimization toolbox that implements the eddy-current nulling constraint and the time-optimal waveform search) — github.com/cmr-group/gropt
- **C** (Language of the optimization routines (src/optimize_kernel.c, minTE_diff) that the eddy-current constraints are applied within) — github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into a Python module so eddy_params and moments_params can be passed from Python) — github.com/cmr-group/gropt
- **Python** (Wrapper used to configure constraints and run the optimization workflows) — github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace
```

## Evaluation signals

- The optimization converges to a feasible waveform (does not hit the 20,000-iteration stop criterion) for each nulling configuration
- Recorded compute times (mean of 100 single-threaded, <10 MB calls) fall within the reported 1-40 ms acceptable range
- eddy_params arrays follow the required format of 4 values per eddy-current constraint; moments_params use 7 values per moment constraint
- Nulling configurations with λ = 80 ms and λ = 40, 80 ms all suppress their specified eddy-current time-constant responses within tolerance ~1.0e-4
- A trade-off between computation time and gradient raster time (dt) is observed as expected when varying dt

## Limitations

- If no feasible solution exists or cannot be found, the optimization stops after 20,000 iterations — nulling constraints may make some configurations infeasible
- With multiple constraints added to the gradient waveform design problem, convexity of the solution space becomes difficult to prove (randomized-start tests are not a complete proof)
- Choice of waveform size N and raster time dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive

## Evidence

- [readme] Fast numerical optimizations for MR gradient waveform design (typically 1-100ms).: [evidence span withheld — anchor status: external_doc]
