---
name: diffusion-b-value-encoding-design
description: Use when a diffusion protocol (e.g. b=1000 mm2/s, six-direction tensor
  encoding) needs a gradient waveform that meets Gmax/Smax hardware limits and moment
  nulling on a specific scanner, or when waveforms must be reproduced across GE and
  Siemens platforms.
when_to_use_negative:
- The task is optimizing readout trajectories or slice-select gradients, which the
  GrOpt toolbox has not been used for
- The goal is a constraint outside the implemented set, such as concomitant field
  terms, acoustic noise, or gradient heating limits (listed as future work)
- The waveform is so long (e.g. multiecho turbo-spin echo) that the default choice
  of N and dt must be redesigned, rather than a standard DWI b-value encoding
edam_operation: http://edamontology.org/operation_0273
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: GrOpt
  role: Time-optimal diffusion waveform design from user-defined constraints (Gmax/Smax,
    b-value, M0/M1/M2 nulling) via an ADMM-based solver
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Core optimization kernels written in C for maximum compatibility on multiple
    scanner systems
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to invoke the compiled GrOpt kernels and compute b-value, moments,
    and duration from G_out/N_out
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines so the Python module can be built with setup.py
    build_ext --inplace
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Alternative wrapper for prototyping and visualizing gradient waveforms
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:0a3c2328ae3e14f12915289b00d61a4e46977a188b80e4ef4cb5e0d3af9b3c25
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
  edam_operation: http://edamontology.org/operation_0273
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# diffusion-b-value-encoding-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design time-optimal diffusion-encoding gradient waveforms with GrOpt that achieve a target b-value (e.g. 1000 mm2/s) under vendor-specific hardware limits (Gmax, Smax, scanner raster time dt) and gradient moment nulling constraints (M0, M1, M2), so that waveforms are functionally equivalent across scanner vendors.

## When to use

Use when a diffusion protocol (e.g. b=1000 mm2/s, six-direction tensor encoding) needs a gradient waveform that meets Gmax/Smax hardware limits and moment nulling on a specific scanner, or when waveforms must be reproduced across GE and Siemens platforms.

## When NOT to use

- The task is optimizing readout trajectories or slice-select gradients, which the GrOpt toolbox has not been used for
- The goal is a constraint outside the implemented set, such as concomitant field terms, acoustic noise, or gradient heating limits (listed as future work)
- The waveform is so long (e.g. multiecho turbo-spin echo) that the default choice of N and dt must be redesigned, rather than a standard DWI b-value encoding

## Inputs

- Diffusion protocol parameters: b-value (e.g. 1000 mm2/s), encoding directions (e.g. six-direction tensor)
- Scanner hardware constraints: Gmax and Smax for the target platform (GE MR750/EPIC, Siemens Skyra/IDEA)
- Gradient raster time dt (set to the scanner gradient raster time) and waveform sample count N
- Moment nulling constraints M0, M1, M2 with residual-moment tolerance

## Outputs

- Diffusion-encoding gradient waveform G_out with sample count N_out
- Achieved diffusion b-value (target 1000 mm2/s)
- Gradient moments M0/M1/M2 and waveform duration T = N*dt
- Per-vendor property comparison (moments, b-value, duration), e.g. vendor_waveform_comparison.csv

## How to apply

First build the GrOpt Python module from the C kernels via Cython (`python setup.py build_ext --inplace` in the python/ directory) to obtain the importable 'gropt' module. For each scanner platform (e.g. GE MR750/EPIC DV26.0R01, Siemens Skyra/IDEA VE11C), configure the hardware constraints Gmax and Smax, the diffusion protocol parameters (b-value 1000 mm2/s, six-direction tensor encoding), and dt equal to the scanner gradient raster time. Run the time-optimal diffusion design (minTE_diff / run_kernel_diff) with moment nulling constraints M0, M1, M2 using the ADMM-based solver; if no feasible solution is found within 20,000 iterations the optimization is stopped. From the output waveform (G_out, N_out) compute the gradient moments M0/M1/M2, the achieved b-value, and duration T = N*dt, and compare these properties across vendors to confirm functional equivalence.

## Related tools

- **GrOpt** (Time-optimal diffusion waveform design from user-defined constraints (Gmax/Smax, b-value, M0/M1/M2 nulling) via an ADMM-based solver) — https://github.com/cmr-group/gropt
- **C** (Core optimization kernels written in C for maximum compatibility on multiple scanner systems) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used to invoke the compiled GrOpt kernels and compute b-value, moments, and duration from G_out/N_out) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines so the Python module can be built with setup.py build_ext --inplace) — https://github.com/cmr-group/gropt
- **Matlab** (Alternative wrapper for prototyping and visualizing gradient waveforms) — https://github.com/cmr-group/gropt

## Examples

```
python setup.py build_ext --inplace; import gropt; G_out, N_out = gropt.minTE_diff(bvalue=1000, M0=0, M1=0, M2=0, dt=5e-3, Gmax=4, Smax=15)
```

## Evaluation signals

- Each generated waveform meets its moment constraints (M0, M1, M2) within the specified residual-moment tolerance
- The achieved diffusion b-value equals the protocol target (1000 mm2/s)
- Waveform duration T = N*dt is feasible, with no feasible solution accepted only after the 20,000-iteration stopping criterion
- Per-vendor waveform properties (moments, b-value, duration) assembled in vendor_waveform_comparison.csv show functional equivalence between GE and Siemens designs
- Images reconstructed from waveforms run on the different scanners are comparable

## Limitations

- If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations — feasibility is not guaranteed
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- With multiple constraints, convexity of the solution space is difficult to prove, and randomized-start tests are not a complete proof

## Evidence

- [methods] ADMM solver basis: [evidence span withheld — anchor status: not_found]
- [readme] building the Python module: [evidence span withheld — anchor status: external_doc]
