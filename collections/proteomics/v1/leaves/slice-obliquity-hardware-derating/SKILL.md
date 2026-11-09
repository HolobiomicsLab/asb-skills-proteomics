---
name: slice-obliquity-hardware-derating
description: Use when designing GrOpt gradient waveforms targeting an oblique slice
  and you must decide between derating Gmax/Smax by a rotation factor versus rotating
  constraints into the physical gradient coordinate system for the specific slice
  orientation.
when_to_use_negative:
- The target slice is axial (no obliquity), so no rotation factor or constraint rotation
  is needed and full hardware limits can be used directly
- Readout trajectories or slice-select gradients are the design target — the article
  notes GrOpt has not been used to optimize readout trajectories or slice-select gradients
- Only a single fixed scanner orientation matters and the waveform never needs to
  be reused across arbitrary scan-plane rotations
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3444
tools:
- name: GrOpt
  role: ADMM-based optimizer that solves the time-optimal gradient waveform feasibility
    problem under derated (invariant) or rotated (variant) constraints
  repo: github.com/cmr-group/gropt
- name: C
  role: Language of the GrOpt optimization kernels for maximum compatibility on multiple
    scanner systems
  repo: github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into a Python module built via setup.py
    build_ext --inplace
  repo: github.com/cmr-group/gropt
- name: Python
  role: Wrapper environment for prototyping, solving, and plotting the waveform pairs
    from both obliquity approaches
  repo: github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:ce9d7f5769c31d96efd92cc936d0416942ee68aefc566f8147efbc1e0e410b70
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# slice-obliquity-hardware-derating

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles oblique slice orientations in time-optimal gradient waveform design with GrOpt by either derating hardware limits (rotationally invariant) or rotating constraints into physical coordinates (rotationally variant), ensuring logical-axis waveforms remain feasible on any scanner rotation.

## When to use

Use when designing GrOpt gradient waveforms targeting an oblique slice and you must decide between derating Gmax/Smax by a rotation factor versus rotating constraints into the physical gradient coordinate system for the specific slice orientation.

## When NOT to use

- The target slice is axial (no obliquity), so no rotation factor or constraint rotation is needed and full hardware limits can be used directly
- Readout trajectories or slice-select gradients are the design target — the article notes GrOpt has not been used to optimize readout trajectories or slice-select gradients
- Only a single fixed scanner orientation matters and the waveform never needs to be reused across arbitrary scan-plane rotations

## Inputs

- Gradient design problem specification: moment constraints, Gmax, Smax, gradient raster time dt, waveform length N
- Oblique slice orientation (rotation matrix or direction cosines for the target slice)
- GrOpt optimizer configuration (ADMM-based feasibility/objective solve, stopping at 20,000 iterations if infeasible)

## Outputs

- Time-optimal gradient waveforms per logical axis (rotationally invariant, derated limits) and per physical axis (rotationally variant, full limits)
- Side-by-side comparison figure annotating the derating factor used
- Feasibility check confirming moment constraints are met by both waveform sets

## How to apply

Define the gradient design problem (e.g., a moment-constrained waveform) with fixed Gmax, Smax, raster time dt, length N, and the target oblique slice orientation, then solve with the ADMM-based GrOpt optimizer. For the rotationally invariant path, reduce Gmax and Smax by the rotation factor (e.g., derating by 3) so the logical-axis waveforms are compatible with all scan-plane rotations. For the rotationally variant path, rotate the relevant constraints into the physical gradient coordinate system for the chosen slice orientation and re-solve at full, underated hardware limits. Plot the resulting waveform pairs (per logical axis and per physical axis) side by side, annotating the derating factor, and export the figure. Validate that both approaches yield feasible waveforms meeting the moment constraints, with the variant waveforms exploiting full hardware limits while the invariant waveforms stay within derated bounds.

## Related tools

- **GrOpt** (ADMM-based optimizer that solves the time-optimal gradient waveform feasibility problem under derated (invariant) or rotated (variant) constraints) — github.com/cmr-group/gropt
- **C** (Language of the GrOpt optimization kernels for maximum compatibility on multiple scanner systems) — github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into a Python module built via setup.py build_ext --inplace) — github.com/cmr-group/gropt
- **Python** (Wrapper environment for prototyping, solving, and plotting the waveform pairs from both obliquity approaches) — github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace  # then in Python: import gropt; g_inv = gropt.gropt(Gmax=4/3, Smax=15/3, moments=[0,0,0], dt=4e-6, N=100)  # invariant: derate Gmax/Smax by 3
```

## Evaluation signals

- Both approaches produce feasible waveforms that satisfy the moment constraints within GrOpt's convergence tolerance
- Rotationally invariant waveforms stay within the derated bounds (Gmax and Smax reduced by the rotation factor of 3) on the logical axes
- Rotationally variant waveforms use the full, underated hardware limits in the physical coordinate system
- Conversing logical-axis waveforms to physical axes for arbitrary rotations remains feasible for the invariant solution
- Optimization terminates within the iteration budget (20,000 iterations) rather than failing to find a feasible solution

## Limitations

- The rotationally invariant approach sacrifices hardware performance (derating by 3) in exchange for guaranteed compatibility with all scan-plane rotations
- The rotationally variant approach must be re-solved for each specific slice orientation since constraints are rotated into a fixed physical coordinate system
- With multiple constraints, convexity of the solution space is difficult to prove, so feasibility is not guaranteed for every obliquity/constraint combination
- Choice of N and dt may need adjustment for longer gradient waveforms

## Evidence

- [methods] Slice obliquity is handled using either a "rotationally invariant" approach that derates the hardware limitations by 3: [evidence span withheld — anchor status: not_found]
- [methods] or using the "rotationally variant" approach that rotates the relevant constraints and solves for optimized gradient waveforms: [evidence span withheld — anchor status: not_found]
- [readme] The optimization routines have been wrapped with Cython, they can be built simply running in the python/ directory: [evidence span withheld — anchor status: external_doc]
- [methods] The GrOpt routine is based on the alternating direction method of multipliers (ADMM), with modifications made to handle any number of constraints: [evidence span withheld — anchor status: not_found]
