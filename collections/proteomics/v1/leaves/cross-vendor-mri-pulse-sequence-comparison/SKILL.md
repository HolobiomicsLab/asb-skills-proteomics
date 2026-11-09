---
name: cross-vendor-mri-pulse-sequence-comparison
description: Use when a diffusion or flow encoding protocol must run comparably on
  scanners from different vendors (e.g., GE and Siemens) and you need to prove waveform-level
  functional equivalence rather than just image-level similarity.
when_to_use_negative:
- You need optimized readout trajectories or slice-select gradients — GrOpt has not
  been used for these
- The goal is waveform design for a single scanner only, without any cross-vendor
  equivalence claim
- You require constraints not yet implemented (e.g., concomitant field terms, acoustic
  noise, gradient heating limits)
tools:
- name: GrOpt
  role: Time-optimal gradient waveform optimizer run per vendor with ADMM solver and
    moment nulling constraints
  repo: github.com/cmr-group/gropt
- name: C
  role: Core optimization kernels written in C for maximum compatibility on multiple
    scanner systems
- name: Cython
  role: Wraps the C optimization routines to produce the importable Python 'gropt'
    module
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to prototype, run waveform optimization, and compute per-vendor
    comparison metrics
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:7fde65d9fa411284a880801b3542cfa022a32dbe7e51092af2ede482620d41d0
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
yes_rate: 0.8571428571428571
n_claims: 14
---

# cross-vendor-mri-pulse-sequence-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates functionally equivalent MRI gradient waveforms on scanners from different vendors by parameterizing a single set of encoding constraints (Gmax, Smax, dt, b-value, moment nulling) per platform with GrOpt, then comparing the realized waveforms on moment, b-value, and duration metrics.

## When to use

Use when a diffusion or flow encoding protocol must run comparably on scanners from different vendors (e.g., GE and Siemens) and you need to prove waveform-level functional equivalence rather than just image-level similarity.

## When NOT to use

- You need optimized readout trajectories or slice-select gradients — GrOpt has not been used for these
- The goal is waveform design for a single scanner only, without any cross-vendor equivalence claim
- You require constraints not yet implemented (e.g., concomitant field terms, acoustic noise, gradient heating limits)

## Inputs

- Per-vendor hardware constraints (Gmax, Smax) from scanner platform specifications
- Diffusion protocol parameters (b-value 1000 mm2/s, six-direction tensor encoding, dt at scanner gradient raster time)
- Moment nulling constraint specification (M0, M1, M2) with residual-moment tolerance

## Outputs

- Per-vendor gradient waveforms (G_out, N_out) runnable on each scanner
- Computed waveform properties: gradient moments M0/M1/M2, achieved b-value, duration T = N*dt
- Cross-vendor property comparison table (vendor_waveform_comparison.csv)

## How to apply

First build the GrOpt Python module from the C kernels with Cython (`python setup.py build_ext --inplace` from the python/ directory). For each vendor platform — in the source work, GE MR750/EPIC DV26.0R01 and Siemens Skyra/IDEA VE11C — set the vendor-specific hardware constraints (Gmax, Smax) plus shared protocol parameters: b-value 1000 mm2/s, six-direction tensor diffusion encoding, and dt equal to the scanner gradient raster time. Run the time-optimal diffusion design (minTE_diff / run_kernel_diff) with moment nulling constraints (M0, M1, M2) using the ADMM-based solver, allowing up to 20,000 iterations if no feasible solution is found; the process is stopped after 20,000 iterations. From each output waveform (G_out, N_out), compute the gradient moments M0/M1/M2, the achieved b-value, and duration T = N*dt, then assemble a per-vendor property comparison (e.g., vendor_waveform_comparison.csv). Validation: each waveform must meet its moment constraints within the specified residual-moment tolerance and achieve b = 1000 mm2/s, establishing the two vendors' waveforms as functionally equivalent.

## Related tools

- **GrOpt** (Time-optimal gradient waveform optimizer run per vendor with ADMM solver and moment nulling constraints) — github.com/cmr-group/gropt
- **C** (Core optimization kernels written in C for maximum compatibility on multiple scanner systems)
- **Cython** (Wraps the C optimization routines to produce the importable Python 'gropt' module) — github.com/cmr-group/gropt
- **Python** (Wrapper used to prototype, run waveform optimization, and compute per-vendor comparison metrics) — github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace && python -c "import gropt; G_out, N_out = gropt.run_kernel_diff(b=1000, dt=6e-6, M0=0, M1=0, M2=0, gmax=0.05, smax=100)"
```

## Evaluation signals

- Each generated waveform satisfies its M0/M1/M2 moment constraints within the specified residual-moment tolerance
- Each waveform achieves the target diffusion b-value of 1000 mm2/s on both vendor platforms
- Waveform duration T = N*dt is consistent across vendors after accounting for per-vendor Gmax/Smax and raster time
- Images reconstructed from the waveforms run on each scanner are comparable
- If no feasible solution is found, the solver is stopped at 20,000 iterations rather than returning an unvalidated waveform

## Limitations

- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- GrOpt has not been used to optimize readout trajectories or slice-select gradients

## Evidence

- [readme] The setup.py file will build the python module. To build seperately you can run python setup.py build_ext --inplace: [evidence span withheld — anchor status: not_found]
