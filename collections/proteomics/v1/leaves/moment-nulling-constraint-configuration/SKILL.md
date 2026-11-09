---
name: moment-nulling-constraint-configuration
description: Use when designing MRI gradient waveforms that must null specific gradient
  moments (flow/velocity, acceleration, or higher-order motion compensation) subject
  to hardware limits (Gmax, slew) and optionally a Schulte et al PNS threshold, e.g.
  phase-contrast flow-encoding gradients.
when_to_use_negative:
- Goal is to optimize readout trajectories or fully free slice-select gradients —
  the paper notes GrOpt 'has not been used to optimize readout trajectories or slice-select
  gradients'
- Required constraints are not yet implemented, e.g. concomitant field terms, acoustic
  noise, or gradient heating limits (listed as future work)
- Waveforms are much longer than the tested regime (e.g. multiecho turbo-spin echo),
  where the default N and dt choice may need re-tuning rather than the standard configuration
tools:
- name: GrOpt
  role: ADMM-based time-optimal gradient waveform optimizer whose modular constraint
    system provides moment nulling, PNS, and eddy-current constraints
  repo: github.com/cmr-group/gropt
- name: C
  role: Core optimization routines written in C, benchmarked for compute time on a
    desktop CPU
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C kernels into the Python module built via setup.py
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper used for prototyping and visualizing the constrained waveforms
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:aa8ded41f011f5f28055c1eac61d28ccc9f6915180179f3d4923b8cbb17e45c2
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# moment-nulling-constraint-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure gradient moment nulling constraints (zeroth through higher order M→n) within the GrOpt time-optimal waveform optimizer so that phase-contrast flow-encoding and other motion-sensitive waveforms nully specified gradient moments while respecting Gmax, slew-rate, and PNS limits.

## When to use

Use when designing MRI gradient waveforms that must null specific gradient moments (flow/velocity, acceleration, or higher-order motion compensation) subject to hardware limits (Gmax, slew) and optionally a Schulte et al PNS threshold, e.g. phase-contrast flow-encoding gradients.

## When NOT to use

- Goal is to optimize readout trajectories or fully free slice-select gradients — the paper notes GrOpt 'has not been used to optimize readout trajectories or slice-select gradients'
- Required constraints are not yet implemented, e.g. concomitant field terms, acoustic noise, or gradient heating limits (listed as future work)
- Waveforms are much longer than the tested regime (e.g. multiecho turbo-spin echo), where the default N and dt choice may need re-tuning rather than the standard configuration

## Inputs

- Hardware constraint set: maximum gradient amplitude (Gmax) and maximum slew rate (Smax)
- List of gradient moments M→n (order n) to be nulled, in mT·ms^n/m·m units
- Optional PNS threshold (Schulte et al single-exponential model) or slew-rate derating factor for slice obliquity
- Waveform length N and raster time dt (paper warns the N/dt choice may need adjustment for longer waveforms such as multiecho turbo-spin echo)

## Outputs

- Time-optimal gradient waveform (sampled at dt) satisfying all moment nulling and hardware constraints within tolerance
- Minimum feasible gradient duration T for the given constraint configuration
- Average per-optimization compute time (benchmark, e.g. mean of 100 calls on a desktop CPU)

## How to apply

Build the GrOpt C kernels and expose them through the Cython Python wrapper (python setup.py build_ext --inplace from the python/ directory). Define the problem by specifying maximum gradient amplitude (Gmax), maximum slew rate (Smax), and the set of gradient moments M→n to be nulled — GrOpt exposes 'constraints for any higher order gradient moment M→n,mTmsnm, which allows for full or partial moment nulling to reduce error due to motion'. For a phase-contrast slice-select/flow-encoding problem, hold the slice-select gradient fixed and flat (excluding intermediate ramps) and add the flow-encoding moment constraints to be nulled. Run the outer time-optimal search over waveform durations T in a multi-threaded fashion, warm-starting short-dt targets from higher-dt solutions; optionally enable the PNS constraint (Schulte et al single-exponential model via PNS_thresh) or slew-rate derating for obliquity. Acceptance is per-constraint: the optimizer 'seeks a gradient waveform subject to the constraints within a specified tolerance', and acceptable waveforms should generate within the reported 1–40 ms compute range on a desktop CPU.

## Related tools

- **GrOpt** (ADMM-based time-optimal gradient waveform optimizer whose modular constraint system provides moment nulling, PNS, and eddy-current constraints) — github.com/cmr-group/gropt
- **C** (Core optimization routines written in C, benchmarked for compute time on a desktop CPU) — github.com/cmr-group/gropt
- **Cython** (Wraps the C kernels into the Python module built via setup.py) — github.com/cmr-group/gropt
- **Python** (Wrapper used for prototyping and visualizing the constrained waveforms) — github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace
```

## Evaluation signals

- Each optimized waveform satisfies its moment and hardware (Gmax, slew, PNS) constraints within the specified tolerance
- Per-waveform compute times fall within the paper's reported 1–40 ms range for acceptable waveform generation
- The outer duration search returns a stable minimum T (e.g. T = 3.42 ms PNS-constrained and T = 4.14 ms slew-rate derated for the phase-contrast case), insensitive to the choice of N
- Adding moment or PNS constraints only lengthens T relative to the unconstrained configuration, never shortens it
- Optimization terminates without hitting the 20,000-iteration infeasibility stop, indicating a feasible solution was found

## Limitations

- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof of convexity
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- If no feasible solution exists or is not found, the optimization stops after 20,000 iterations
- Residual moments due to non-zero convergence criteria can cause intravoxel phase dispersion (assessed via numerical simulation in the paper)

## Evidence

- [readme] Modular constraint system: [evidence span withheld — anchor status: external_doc]
