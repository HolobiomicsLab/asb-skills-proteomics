---
name: pns-constraint-waveform-design
description: Use when a gradient waveform design task (e.g., phase-contrast slice-selection
  or flow encoding, diffusion encoding with moment nulling) must additionally satisfy
  a peripheral nerve stimulation limit, per Schulte et al or Hebrank et al models,
  in GrOpt.
when_to_use_negative:
- Designing readout trajectories or slice-select gradients — GrOpt has not been used
  to optimize those per the discussion
- When the waveform has no patient-comfort requirement — plain moment-nulling or eddy-current-nulling
  designs avoid the O(n^2.32) runtime penalty of the PNS constraint
- When a formal proof of time-optimality or convexity is required — with multiple
  constraints (PNS plus others) convexity is difficult to prove and brute-force optimality
  checks are computationally very expensive
tools:
- name: GrOpt
  role: ADMM-based optimizer whose modular constraint system hosts the Schulte/Hebrank
    PNS constraint during time-optimal waveform design
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the core optimization routines, for native integration on scanner
    systems
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C routines into a Python module built with setup.py build_ext --inplace
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to prototype and run PNS-constrained waveform optimizations (Jupyter
    demos in ./python/)
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Wrapper for prototyping/visualizing PNS-constrained waveforms via make.m compiled
    mex functions
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:3ccb3a0280676c2cb1252cf81e2580c9457c771721663c4371ac85b8475b3a31
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pns-constraint-waveform-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design time-optimal MRI gradient waveforms in GrOpt while explicitly limiting peripheral nerve stimulation (PNS), using the Schulte et al or Hebrank et al PNS models as constraints inside the ADMM-based optimization. Adding the PNS constraint raises compute-time scaling relative to unconstrained waveform design, so this skill trades runtime for patient comfort.

## When to use

Use when a gradient waveform design task (e.g., phase-contrast slice-selection or flow encoding, diffusion encoding with moment nulling) must additionally satisfy a peripheral nerve stimulation limit, per Schulte et al or Hebrank et al models, in GrOpt.

## When NOT to use

- Designing readout trajectories or slice-select gradients — GrOpt has not been used to optimize those per the discussion
- When the waveform has no patient-comfort requirement — plain moment-nulling or eddy-current-nulling designs avoid the O(n^2.32) runtime penalty of the PNS constraint
- When a formal proof of time-optimality or convexity is required — with multiple constraints (PNS plus others) convexity is difficult to prove and brute-force optimality checks are computationally very expensive

## Inputs

- Set of sequence constraints defining the acquisition (encoding targets, gradient moment nulling orders, eddy-current nulling time constants, hardware Gmax/slew limits)
- Choice of PNS model (Schulte et al or Hebrank et al) with its threshold parameters
- Gradient waveform size N and scanner raster time dt
- Built GrOpt C library (src/) or compiled Python/Cython module

## Outputs

- Time-optimal gradient waveform (sampled time series g(t)) satisfying the PNS constraint
- Minimum feasible waveform duration T / minimum TE for the constrained design
- Convergence/feasibility status and, if benchmarked, per-N compute times for scaling analysis

## How to apply

Build the GrOpt C optimization routines via the Cython/Python wrapper (`python setup.py build_ext --inplace` in the python/ directory) and supply the PNS model (Schulte et al or Hebrank et al, both implemented in GrOpt as constraints) alongside the application constraints (e.g., gradient moment nulling, eddy-current nulling, hardware Gmax/slew limits). The ADMM-based routine searches waveform durations T — using a coarse high-dt search to warm start a finer solution — seeking a feasible waveform within a specified tolerance, stopping after 20,000 iterations if no feasible solution is found. Expect a runtime penalty from the PNS constraint: for phase-contrast optimization compute time scales as O(n^1.50) without PNS but O(n^2.32) with it, so benchmark ~100 averaged invocations per waveform size N if timing matters. Accept the design when the generated waveform is feasible under all supplied constraints and generation falls in the reported 1-40 ms range (1-100 ms per the README).

## Related tools

- **GrOpt** (ADMM-based optimizer whose modular constraint system hosts the Schulte/Hebrank PNS constraint during time-optimal waveform design) — https://github.com/cmr-group/gropt
- **C** (Language of the core optimization routines, for native integration on scanner systems) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C routines into a Python module built with setup.py build_ext --inplace) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used to prototype and run PNS-constrained waveform optimizations (Jupyter demos in ./python/)) — https://github.com/cmr-group/gropt
- **Matlab** (Wrapper for prototyping/visualizing PNS-constrained waveforms via make.m compiled mex functions) — https://github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace
```

## Evaluation signals

- The returned waveform satisfies the Schulte/Hebrank PNS constraint within the solver's specified tolerance alongside all other supplied constraints
- Waveform generation time falls in the reported 1-40 ms range (README: typically 1-100 ms), confirming on-the-fly viability despite the PNS constraint
- Benchmarked scaling with waveform size N shows the expected penalty: exponent increases from ~1.50 (no PNS) to ~2.32 (with PNS) for phase-contrast designs
- The optimization converges before the 20,000-iteration stop; hitting the cap indicates no feasible PNS-compliant solution exists for the given constraints/duration
- Multi-scanner reproduction: waveforms generated from the same constraint set are functionally equivalent across vendor platforms (GE EPIC, Siemens IDEA)

## Limitations

- With multiple constraints (e.g., PNS plus moment nulling), convexity of the solution space is difficult to prove; randomized-start tests are not a complete proof
- If no feasible PNS-compliant solution exists or is found, the optimizer stops after 20,000 iterations without a waveform
- The default choice of gradient size (N) and raster time (dt) may need adjustment for longer gradient waveforms, and compute time with the PNS constraint scales more steeply (O(n^2.32))
- README reports no changelog/version information for the toolbox, so reproducing exact published timings may require the nmrb_optimization branch or the paper's referenced codebase

## Evidence

- [readme] Typical runtime window: [evidence span withheld — anchor status: external_doc]
