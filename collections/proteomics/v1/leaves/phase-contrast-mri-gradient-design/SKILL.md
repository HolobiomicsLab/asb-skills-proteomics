---
name: phase-contrast-mri-gradient-design
description: Use when a phase-contrast MRI pulse design must null flow-encoding moments
  in minimum time T, and the dependence of minimum T and compute time on raster time
  dt and added PNS/moment constraints is of interest.
when_to_use_negative:
- The task is optimizing readout trajectories or non-phase-contrast slice-select gradients
  — the article notes the GrOpt toolbox has not been used to optimize readout trajectories
  or slice-select gradients in general
- Longer gradient waveforms such as multiecho turbo-spin-echo sequences are targeted
  — the article notes the choice of N and dt may need adjustment there
- The design goal is b-value maximization or duty-cycle minimization for diffusion
  rather than flow-encoding moment nulling for phase contrast
tools:
- name: GrOpt
  role: Core time-optimal gradient waveform optimizer (ADMM-based) run for each duration
    T and constraint configuration
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language in which the optimization routines are implemented for speed and
    scanner compatibility
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into the Python module built via setup.py
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to configure and invoke the phase-contrast waveform optimizations
    and benchmarking
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:b2bf411a630c37dc1c91d84460e48df7c4c9140cc983c9ac34e0c8387760dbae
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# phase-contrast-mri-gradient-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs time-optimal phase-contrast MRI slice-selection and flow-encoding gradient waveforms with GrOpt, searching over waveform durations T and raster times dt under Gmax, slew-rate, gradient-moment-nulling, and optional PNS constraints, and benchmarks compute time per configuration.

## When to use

Use when a phase-contrast MRI pulse design must null flow-encoding moments in minimum time T, and the dependence of minimum T and compute time on raster time dt and added PNS/moment constraints is of interest.

## When NOT to use

- The task is optimizing readout trajectories or non-phase-contrast slice-select gradients — the article notes the GrOpt toolbox has not been used to optimize readout trajectories or slice-select gradients in general
- Longer gradient waveforms such as multiecho turbo-spin-echo sequences are targeted — the article notes the choice of N and dt may need adjustment there
- The design goal is b-value maximization or duty-cycle minimization for diffusion rather than flow-encoding moment nulling for phase contrast

## Inputs

- Gmax and Smax hardware constraint specifications
- Fixed flat slice-select gradient profile (excluding intermediate ramps)
- Flow-encoding gradient moment constraints to be nulled
- Raster time dt values (waveform length N) to sweep
- Optional PNS threshold configuration (Schulte et al single-exponential model)

## Outputs

- Time-optimal phase-contrast slice-select and flow-encoding gradient waveforms per dt and constraint configuration
- Table of minimum gradient duration T versus dt and constraint configuration (e.g. phase_contrast_timing_table.csv)
- Average per-waveform compute time over 100 optimization calls per configuration

## How to apply

Build the GrOpt C optimization kernels and expose them through the Cython-built Python wrapper ('python setup.py build_ext --inplace' in the python/ directory). Configure the problem with maximum gradient amplitude (Gmax), maximum slew rate (Smax), a fixed flat slice-select gradient (excluding intermediate ramps), and flow-encoding moment constraints to be nulled. For each raster time dt and constraint configuration, run the outer time-optimal search over waveform durations T in a multi-threaded fashion, warm-starting the target-dt final optimization from higher-dt solutions. Repeat with a PNS constraint enabled (Schulte et al single-exponential model via PNS_thresh) to measure its effect on duration and compute time. Benchmark by averaging 100 individual optimization calls on a desktop CPU (e.g. Intel Core i7-7700k), then tabulate minimum gradient duration versus average compute time per dt configuration (e.g. phase_contrast_timing_table.csv). The article reports T = 3.42 ms PNS-constrained and T = 4.14 ms slew-rate-derated, with waveform generation in 1-40 ms being acceptable for on-the-fly use.

## Related tools

- **GrOpt** (Core time-optimal gradient waveform optimizer (ADMM-based) run for each duration T and constraint configuration) — https://github.com/cmr-group/gropt
- **C** (Language in which the optimization routines are implemented for speed and scanner compatibility) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into the Python module built via setup.py) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used to configure and invoke the phase-contrast waveform optimizations and benchmarking) — https://github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace
```

## Evaluation signals

- Per-waveform compute times fall within the paper's reported acceptable 1-40 ms range for on-the-fly waveform generation
- Each optimized waveform satisfies its moment, Gmax, and slew-rate constraints within the specified optimization tolerance
- Minimum T is insensitive to the choice of N (expected T ≈ 3.42 ms PNS-constrained, ≈ 4.14 ms slew-rate derated for the paper's configuration)
- Enabling the PNS constraint increases minimum T relative to the slew-rate-only configuration
- Optimization does not terminate via the 20,000-iteration infeasibility stop for the tested configurations

## Limitations

- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof of convexity
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- The choice of gradient size N and raster time dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin-echo sequences

## Evidence

- [readme] This will use Cython to generate the source files for a Python module, and then compile it within the GrOpt folder.: [evidence span withheld — anchor status: external_doc]
