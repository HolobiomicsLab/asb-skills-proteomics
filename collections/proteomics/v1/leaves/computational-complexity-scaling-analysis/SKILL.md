---
name: computational-complexity-scaling-analysis
description: Decision support for determining whether a numerical optimization routine
  (e.g., GrOpt ADMM gradient waveform design) is fast enough for on-the-fly use, and
  for quantifying how constraint sets change its asymptotic compute-time scaling with
  input size N.
when_to_use_negative:
- The goal is validating whether a waveform is truly time-optimal — the article notes
  brute-force verification is computationally very expensive and out of scope here
- The optimizer output quality (e.g., residual gradient moments, intravoxel phase
  dispersion, ADC bias) is the target — that requires the numerical simulation workflow,
  not timing
- The routine's solution space convexity is the question — randomized-start tests
  are not a complete proof, so a scaling benchmark would not address it
edam_operation: http://edamontology.org/operation_2945
tools:
- name: GrOpt
  role: ADMM-based optimizer being benchmarked; its C routines (minTE_diff in src/optimize_kernel.c)
    are timed over varying N
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the optimization routines being compiled and benchmarked for maximum
    compatibility on scanner systems
- name: Python
  role: Wrapper (via Cython) used to invoke the optimization kernels repeatedly and
    script the timing/log-regression fits
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Builds the Python module from the C routines (python setup.py build_ext --inplace)
    so the kernels can be called from timing scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:6b59740943b97a7062216af37a4038816fee0f470a9fa5792f0a6ac2fddf5e9c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# computational-complexity-scaling-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirically measures how an optimizer's compute time scales with problem size (waveform sample count N) by timing repeated optimization runs and fitting a power law t ~ N^alpha, and quantifies how adding constraints (e.g., a Schulte PNS constraint to GrOpt phase-contrast optimization) inflates the fitted exponent.

## When to use

Decision support for determining whether a numerical optimization routine (e.g., GrOpt ADMM gradient waveform design) is fast enough for on-the-fly use, and for quantifying how constraint sets change its asymptotic compute-time scaling with input size N.

## When NOT to use

- The goal is validating whether a waveform is truly time-optimal — the article notes brute-force verification is computationally very expensive and out of scope here
- The optimizer output quality (e.g., residual gradient moments, intravoxel phase dispersion, ADC bias) is the target — that requires the numerical simulation workflow, not timing
- The routine's solution space convexity is the question — randomized-start tests are not a complete proof, so a scaling benchmark would not address it

## Inputs

- GrOpt C source tree from github.com/cmr-group/gropt (src/optimize_kernel.c with minTE_diff kernel)
- Range of gradient waveform sample counts N at raster time dt for each application (diffusion M0/M1/M2 moment nulling and eddy-current nulling variants; phase-contrast slice-selection and flow encoding)
- Constraint-set configuration flags: with and without the Schulte PNS constraint (also Hebrank model available)
- Single-threaded benchmark host specification (reference: Intel Core i7-7700k, <10 Mb memory)

## Outputs

- Timing table of mean compute time per (application, constraint-set, N) combination
- compute_time_scaling_exponents.csv with one fitted exponent alpha per application/constraint configuration
- Fitted power-law models t ~ N^alpha per configuration

## How to apply

Build the C optimization routines (GrOpt, wrapped with Cython for Python) and run the target kernel (e.g., minTE_diff/run_kernel_diff for phase-contrast slice-selection and flow encoding gradients, with and without the Schulte PNS constraint) over a range of waveform sample counts N at the scanner raster time dt, averaging 100 invocations per configuration on a single-threaded desktop (reference: Intel Core i7-7700k, <10 Mb memory). Log mean compute time per (application, constraint-set, N) combination into a timing table. Fit t ~ N^alpha by linear regression on log(compute time) versus log(N) for each configuration, and assemble the exponents into a CSV (e.g., compute_time_scaling_exponents.csv) with one row per application/constraint configuration. Compare the fitted exponents against the paper's reported complexity range and reported runtimes to judge success.

## Related tools

- **GrOpt** (ADMM-based optimizer being benchmarked; its C routines (minTE_diff in src/optimize_kernel.c) are timed over varying N) — https://github.com/cmr-group/gropt
- **C** (Language of the optimization routines being compiled and benchmarked for maximum compatibility on scanner systems)
- **Python** (Wrapper (via Cython) used to invoke the optimization kernels repeatedly and script the timing/log-regression fits) — https://github.com/cmr-group/gropt
- **Cython** (Builds the Python module from the C routines (python setup.py build_ext --inplace) so the kernels can be called from timing scripts)

## Examples

```
cd python && python setup.py build_ext --inplace; then time gropt minTE_diff over varying N and fit alpha, _ = np.polyfit(np.log(N), np.log(t_mean), 1)
```

## Evaluation signals

- Fitted exponents for the unconstrained phase-contrast configuration and PNS-constrained configuration reproduce the card-reported scaling (O(n^1.50) without PNS; O(n^2.32) with PNS constraint)
- Fitted exponents fall within the article's reported complexity range of O(n^0.90) to O(n^1.32) for the benchmarked application set
- Benchmarked per-waveform runtimes fall in the reported 1-40 ms band (README states 1-100 ms typically)
- Each timing cell is the mean of 100 repeated invocations, and the log-log fit residuals show a stable linear power-law relationship
- compute_time_scaling_exponents.csv contains exactly one row per application/constraint configuration tested

## Limitations

- Hardware-dependent: results are calibrated against a single-threaded Intel Core i7-7700k desktop with <10 Mb memory; other hosts may shift absolute times
- The choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Compute times do not establish true time-optimality of the waveforms, and convexity with multiple constraints is difficult to prove
- The GrOpt toolbox has no changelog or version information, so the exact code revision benchmarked should be pinned manually

## Evidence

- [methods] Compute times were tested for a range of pulse sequence applications using a range of gradient waveform constraints: "Compute times were tested for a range of pulse sequence applications using a range of gradient waveform constraints"
- [abstract] it was observed that acceptable gradient waveforms could be generated in 1-40 ms: "it was observed that acceptable gradient waveforms could be generated in 1-40 ms."
- [methods] The GrOpt routine is based on ADMM: [evidence span withheld — anchor status: not_found]
- [readme] Fast numerical optimizations for MR gradient waveform design (typically 1-100ms).: [evidence span withheld — anchor status: external_doc]
