---
name: mri-gradient-moment-nulling
description: Use when designing MRI gradient waveforms that must be fully or partially
  moment-nulled (e.g., b=1000 mm2/s six-direction diffusion encoding on GE MR750/EPIC
  or Siemens Skyra/IDEA) within specific Gmax/Smax hardware limits and a time-optimal
  duration.
when_to_use_negative:
- Optimizing readout trajectories or slice-select gradients — the article states the
  GrOpt toolbox has not been used to optimize these
- Designs requiring constraint types not yet included in the toolbox, such as concomitant
  field terms, acoustic noise, or gradient heating limits
- When an existing validated vendor-supplied waveform already meets moment and b-value
  requirements and no time-optimal redesign is needed
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: GrOpt
  role: Core optimizer that finds time-optimal gradient waveforms subject to moment
    nulling (M0/M1/M2) and hardware constraints via ADMM
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the optimization kernels in src/ for maximum compatibility and
    native integration on multiple scanner systems
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into an importable Python module during
    build
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to build, prototype, and run the diffusion waveform design (e.g.,
    minTE_diff / run_kernel_diff) with Jupyter notebook demos in the python/ folder
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:d837a0f28233aee6c26ba6899532fd5b62cdbf7d6a6ad16e1ee8b1367d6101e1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mri-gradient-moment-nulling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design time-optimal MRI diffusion/flow-encoding gradient waveforms with gradient moment nulling constraints (M0, M1, M2 and higher orders) using the GrOpt toolbox, so motion-induced intravoxel phase dispersion is suppressed. Waveforms are solved via ADMM under vendor-specific hardware limits (Gmax, Smax) and validated by residual moments and achieved b-value.

## When to use

Use when designing MRI gradient waveforms that must be fully or partially moment-nulled (e.g., b=1000 mm2/s six-direction diffusion encoding on GE MR750/EPIC or Siemens Skyra/IDEA) within specific Gmax/Smax hardware limits and a time-optimal duration.

## When NOT to use

- Optimizing readout trajectories or slice-select gradients — the article states the GrOpt toolbox has not been used to optimize these
- Designs requiring constraint types not yet included in the toolbox, such as concomitant field terms, acoustic noise, or gradient heating limits
- When an existing validated vendor-supplied waveform already meets moment and b-value requirements and no time-optimal redesign is needed

## Inputs

- Diffusion protocol parameters: b-value (e.g., 1000 mm2/s), encoding directions (e.g., six-direction tensor)
- Vendor scanner hardware limits: Gmax and Smax
- Gradient raster time dt and waveform sample count N
- Moment nulling constraint specification (M0, M1, M2 or higher order gradient moment)

## Outputs

- Gradient waveform samples (G_out) with N_out time points
- Computed gradient moments M0/M1/M2 of the generated waveform
- Achieved diffusion b-value and waveform duration T = N*dt
- Per-vendor waveform property comparison table (e.g., vendor_waveform_comparison.csv)

## How to apply

Build the GrOpt C optimization kernels as an importable Python module via Cython (python setup.py build_ext --inplace). For each vendor platform, configure hardware constraints Gmax and Smax plus protocol parameters (b-value 1000 mm2/s, six-direction tensor diffusion encoding, dt set to the scanner gradient raster time). Run the time-optimal diffusion design (minTE_diff / run_kernel_diff) with moment nulling constraints (M0, M1, M2) and the ADMM-based solver, allowing up to 20,000 iterations if no feasible solution is found. From each output waveform (G_out, N_out), compute the gradient moments M0/M1/M2, achieved b-value, and duration T = N*dt. Verify each waveform meets its moment constraints within the specified residual-moment tolerance and achieves the target b-value; assemble per-vendor comparisons (moments, b-value, duration) to confirm functional equivalence.

## Related tools

- **GrOpt** (Core optimizer that finds time-optimal gradient waveforms subject to moment nulling (M0/M1/M2) and hardware constraints via ADMM) — https://github.com/cmr-group/gropt
- **C** (Language of the optimization kernels in src/ for maximum compatibility and native integration on multiple scanner systems) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into an importable Python module during build) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used to build, prototype, and run the diffusion waveform design (e.g., minTE_diff / run_kernel_diff) with Jupyter notebook demos in the python/ folder) — https://github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace
```

## Evaluation signals

- Each generated waveform's residual gradient moments M0/M1/M2 fall within the specified residual-moment tolerance
- Achieved diffusion b-value equals the target (1000 mm2/s) for each waveform
- Optimizer converges to a feasible solution before the 20,000-iteration cutoff; failure to converge indicates infeasible constraints
- Per-vendor waveform properties (moments, b-value, duration T = N*dt) are functionally equivalent across GE MR750/EPIC and Siemens Skyra/IDEA, and resulting images are comparable
- Waveform duration matches the time-optimal search result, with dead time eliminated (reduced TE for higher SNR)

## Limitations

- With multiple constraints, convexity of the solution space is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- If no feasible solution exists or cannot be found, the process stops after 20,000 iterations (no waveform is produced)

## Evidence

- [readme] Build command for the Python module via Cython: [evidence span withheld — anchor status: external_doc]
