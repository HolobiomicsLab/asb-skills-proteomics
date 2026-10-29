---
name: rotation-matrix-constraint-transformation
description: Use when designing gradient waveforms with GrOpt for an oblique imaging
  slice, and you must decide between rotationally invariant derating (Gmax/Smax reduced
  by 3) versus rotating constraints into physical coordinates.
when_to_use_negative:
- The imaging plane is axial-aligned with the physical gradient axes (no rotation),
  so derating or constraint rotation is unnecessary and would waste gradient headroom
- The problem is not a GrOpt-style constrained waveform design (e.g., optimizing readout
  trajectories or slice-select gradients, which GrOpt has not been used for)
- You need constraint types not yet implemented in GrOpt (e.g., concomitant field
  terms, acoustic noise, gradient heating limits)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: GrOpt
  role: ADMM-based optimizer that solves the constrained waveform design problem in
    either derated logical coordinates or rotated physical coordinates
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Implementation language of the optimization kernels that apply the rotated
    or derated constraints
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used to define constraints, invoke the optimizer, and plot/logical-vs-physical
    waveforms
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines for building the Python module (python setup.py
    build_ext --inplace)
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:b435a31478b17457ab01dd1e386266905b9abcf822f18610029a014b9de0d66e
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
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rotation-matrix-constraint-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles slice obliquity in GrOpt time-optimal gradient waveform design by either (a) derating hardware limits (Gmax, Smax) by the rotation factor (up to 3) so logical-axis waveforms remain feasible for any scan-plane rotation, or (b) rotating the moment and hardware constraints into the physical gradient coordinate system and solving at full limits.

## When to use

Use when designing gradient waveforms with GrOpt for an oblique imaging slice, and you must decide between rotationally invariant derating (Gmax/Smax reduced by 3) versus rotating constraints into physical coordinates.

## When NOT to use

- The imaging plane is axial-aligned with the physical gradient axes (no rotation), so derating or constraint rotation is unnecessary and would waste gradient headroom
- The problem is not a GrOpt-style constrained waveform design (e.g., optimizing readout trajectories or slice-select gradients, which GrOpt has not been used for)
- You need constraint types not yet implemented in GrOpt (e.g., concomitant field terms, acoustic noise, gradient heating limits)

## Inputs

- Gradient design problem specification: moment constraints (e.g., M0/M1 flow compensation), Gmax and Smax hardware limits, gradient raster time dt, waveform length N
- Target slice orientation / rotation relative to the physical gradient coordinate system
- Choice of obliquity handling mode: rotationally invariant (derated) vs rotationally variant (rotated constraints)

## Outputs

- Time-optimal gradient waveform(s) g(t) satisfying the moment constraints within tolerance
- Waveform pairs per logical axis (invariant/derated) and per physical axis (variant/rotated) suitable for side-by-side comparison

## How to apply

Define the gradient design problem (moment constraints, Gmax, Smax, raster time dt, waveform length N) for a target oblique slice. Choose the rotationally invariant path by dividing the hardware limits (Gmax and Smax) by the rotation factor of 3 so the resulting logical-axis waveforms are feasible under any scan-plane rotation, then solve with the ADMM-based GrOpt optimizer. Alternatively, choose the rotationally variant path: rotate the relevant constraints (moment constraints and hardware limits) into the physical gradient coordinate system for the specific slice orientation, and re-solve at the full, underated hardware limits. Compare the two solutions per logical and per physical axis; the rotationally variant solution should exploit higher effective gradient amplitudes while both must satisfy the moment constraints.

## Related tools

- **GrOpt** (ADMM-based optimizer that solves the constrained waveform design problem in either derated logical coordinates or rotated physical coordinates) — https://github.com/cmr-group/gropt
- **C** (Implementation language of the optimization kernels that apply the rotated or derated constraints) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used to define constraints, invoke the optimizer, and plot/logical-vs-physical waveforms) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines for building the Python module (python setup.py build_ext --inplace)) — https://github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace
```

## Evaluation signals

- Both approaches yield waveforms that are feasible and meet the specified gradient moment constraints within tolerance
- The rotationally invariant waveforms never exceed the derated bounds (Gmax and Smax reduced by the rotation factor of 3)
- The rotationally variant waveforms use the full (underated) hardware limits while remaining feasible in the physical coordinate system
- Optimizer converges before the 20,000-iteration stop threshold (no infeasibility flag)
- Solve time stays in the reported 1-40 ms range for acceptable waveforms

## Limitations

- The rotationally invariant approach sacrifices up to a factor of 3 in gradient amplitude/slew headroom because it must be feasible under any scan-plane rotation
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- Convexity of the solution space with multiple constraints is difficult to prove; randomized-start tests are not a complete proof

## Evidence

- [methods] Slice obliquity handling: two approaches: [evidence span withheld — anchor status: not_found]
- [methods] Rotationally variant approach: [evidence span withheld — anchor status: not_found]
- [methods] Optimization basis: [evidence span withheld — anchor status: not_found]
