---
name: optimization-compute-time-benchmarking
description: Use when quantifying how fast GrOpt produces feasible gradient waveforms
  (diffusion, phase-contrast) under moment, eddy-current, or PNS constraints, and
  how compute time depends on raster time dt or allowed residual moments.
when_to_use_negative:
- The goal is waveform accuracy or image quality (ADC bias from residual moments,
  intravoxel phase dispersion) rather than optimization speed — use a simulation-based
  skill instead.
- The optimization targets readout trajectories or slice-select gradient design, which
  the GrOpt toolbox has not been used to optimize.
- Very long waveform designs (e.g., multiecho turbo-spin echo) where the default choice
  of N and dt may need adjustment and the 1-40 ms benchmark range may not hold.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3444
tools:
- name: GrOpt
  role: ADMM-based optimization toolbox whose optimization calls are benchmarked;
    C kernels run the time-optimal searches (minTE_diff, test_TE_finder)
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the optimization routines that are timed in the 100-call benchmark
    loop
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines for building the Python module used in benchmarking
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper environment (Anaconda, Python 3.7 tested) for running and scripting
    the benchmark configurations
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:b1c9c16f6f51cdc09c7adb1a80b5154a257ffa1f5b937e21a060e9adc860f329
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# optimization-compute-time-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock compute time of GrOpt numerical optimizations (ADMM-based gradient waveform design in C) by averaging repeated single-threaded calls per constraint configuration, to verify on-the-fly (1-40 ms) waveform generation.

## When to use

Use when quantifying how fast GrOpt produces feasible gradient waveforms (diffusion, phase-contrast) under moment, eddy-current, or PNS constraints, and how compute time depends on raster time dt or allowed residual moments.

## When NOT to use

- The goal is waveform accuracy or image quality (ADC bias from residual moments, intravoxel phase dispersion) rather than optimization speed — use a simulation-based skill instead.
- The optimization targets readout trajectories or slice-select gradient design, which the GrOpt toolbox has not been used to optimize.
- Very long waveform designs (e.g., multiecho turbo-spin echo) where the default choice of N and dt may need adjustment and the 1-40 ms benchmark range may not hold.

## Inputs

- GrOpt source tree with C kernels (src/optimize_kernel.c) and python/setup.py for Cython build
- Hardware limits (Gmax in T/m, Smax in T/m/s), waveform size N, and raster time dt
- Constraint configurations: moment_params arrays (7 values per M0/M1/M2 constraint), eddy_params (4 values per time constant, e.g. λ = 40, 80 ms), PNS_thresh for PNS-constrained designs
- Target b-value or flow-encoding moment constraints defining the design problem

## Outputs

- Average compute time per configuration (mean over 100 optimization calls)
- Optimized time-optimal gradient waveforms with their achieved parameters (minimum T, b-value, residual moments)
- CSV tables such as compute_times_diffusion.csv or phase_contrast_timing_table.csv relating compute time to dt and constraint configuration

## How to apply

Build the GrOpt C kernels (src/optimize_kernel.c) and Cython/Python wrapper via 'python setup.py build_ext --inplace', then configure the waveform problem with hardware limits (Gmax, Smax), waveform size N, raster time dt, and the relevant constraints (moment_params with 7 values per M0/M1/M2 constraint; eddy_params with 4 values per time constant, tolerance ~1.0e-4). Run the outer multi-threaded time-optimal duration search (minTE_diff for diffusion) that warm-starts target-dt optimizations from higher-dt solutions. Benchmark each configuration by averaging the duration of 100 individual calls to the optimization function on the C routines, single threaded with minimal memory (<10 Mb), with 20,000 iterations as the failure stop criterion. Tabulate average compute time together with waveform parameters (e.g., compute_times_diffusion.csv or phase_contrast_timing_table.csv) and validate that times fall within the reported 1-40 ms range for acceptable waveforms while constraints are satisfied within tolerance.

## Related tools

- **GrOpt** (ADMM-based optimization toolbox whose optimization calls are benchmarked; C kernels run the time-optimal searches (minTE_diff, test_TE_finder)) — github.com/cmr-group/gropt
- **C** (Language of the optimization routines that are timed in the 100-call benchmark loop) — github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines for building the Python module used in benchmarking) — github.com/cmr-group/gropt
- **Python** (Wrapper environment (Anaconda, Python 3.7 tested) for running and scripting the benchmark configurations) — github.com/cmr-group/gropt

## Examples

```
python setup.py build_ext --inplace
```

## Evaluation signals

- Per-configuration average compute time (over 100 single-threaded C calls) falls within the reported 1-40 ms range for acceptable waveforms.
- Each optimized waveform converges to a feasible solution satisfying its moment, eddy-current (tolerance ~1.0e-4), and hardware constraints within tolerance.
- Optimization that cannot find a feasible solution stops at the 20,000-iteration failure criterion rather than hanging.
- Compute time shows the expected trade-off with gradient raster time dt (finer dt costs more compute time).
- Output CSV contains compute-time and waveform-parameter entries for every tested constraint configuration.

## Limitations

- Compute time depends on gradient raster time dt and waveform length N; the choice of N and dt may need adjustment for longer waveforms such as multiecho turbo-spin echo sequences.
- Timing was performed single-threaded on a specific desktop CPU (Intel Core i7-7700k); absolute times will vary with hardware.
- Benchmarks cover diffusion and phase-contrast designs; GrOpt has not been used to optimize readout trajectories or slice-select gradients.
- No changelog or version information was provided for the toolbox, so benchmarked behavior may differ across repository revisions.

## Evidence

- [readme] Cython build command: [evidence span withheld — anchor status: external_doc]
- [readme] Compute time range in README: [evidence span withheld — anchor status: external_doc]
