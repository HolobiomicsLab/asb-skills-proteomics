---
name: gradient-waveform-comparison-visualization
description: Use when a gradient design task must contrast waveforms solved at derated
  logical-axis limits against waveforms solved at full hardware limits in the physical
  coordinate system for an oblique slice orientation.
when_to_use_negative:
- 'The slice is axial (no obliquity): rotationally invariant derating needlessly sacrifices
  hardware headroom and no comparison is meaningful'
- The task is checking waveform feasibility or timing rather than contrasting obliquity
  handling strategies — use the GrOpt demos in ./python/ notebooks directly instead
- Readout trajectory or slice-select gradient optimization, which the GrOpt toolbox
  has not been used for
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3444
tools:
- name: GrOpt
  role: ADMM-based optimizer that solves the moment-constrained waveform for both
    the derated logical-axis and rotated physical-axis constraint sets
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used for prototyping and visualizing the gradient waveforms (Jupyter
    notebook demos in ./python/)
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Alternative wrapper for prototyping and visualizing gradient waveforms via
    demo_* scripts in ./matlab/
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines into the Python module; built with setup.py
    build_ext --inplace
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:5e703963be8004e1e22f97f7bc8059420c6f27dbeef3fb5a2e51abb0ce1add0a
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gradient-waveform-comparison-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates side-by-side plots of GrOpt-optimized gradient waveforms from the two slice-obliquity handling strategies (rotationally invariant derating vs rotationally variant physical-coordinate solving), annotating the derating factor and exporting the comparison figure.

## When to use

Use when a gradient design task must contrast waveforms solved at derated logical-axis limits against waveforms solved at full hardware limits in the physical coordinate system for an oblique slice orientation.

## When NOT to use

- The slice is axial (no obliquity): rotationally invariant derating needlessly sacrifices hardware headroom and no comparison is meaningful
- The task is checking waveform feasibility or timing rather than contrasting obliquity handling strategies — use the GrOpt demos in ./python/ notebooks directly instead
- Readout trajectory or slice-select gradient optimization, which the GrOpt toolbox has not been used for

## Inputs

- GrOpt gradient design problem specification (moment constraints M0/M1..., Gmax, Smax, raster time dt, waveform length N)
- Oblique slice orientation (rotation defining physical vs logical gradient axes)
- Compiled GrOpt Python or Matlab wrapper from github.com/cmr-group/gropt

## Outputs

- Two feasible sets of gradient waveforms: derated logical-axis waveforms and full-limit physical-axis waveforms
- Annotated side-by-side comparison figure (per logical axis and per physical axis) with the derating factor labeled
- Exported figure file

## How to apply

Build the GrOpt Python wrapper (C kernels compiled via Cython with 'python setup.py build_ext --inplace' from the python/ directory), then define a moment-constrained waveform problem with fixed Gmax/Smax hardware limits, raster time dt, waveform length N, and an oblique slice orientation. Solve the rotationally invariant variant by derating Gmax and Smax by the rotation factor (per-axis derating factor of 3 when hardware limits are assumed identical on all axes) so the logical waveforms remain feasible under all scan-plane rotations, using the ADMM-based GrOpt optimizer. Then re-solve the rotationally variant case by rotating the relevant constraints into the physical gradient coordinate system for the chosen slice orientation at full, underated hardware limits. Plot the resulting waveform pairs per logical axis and per physical axis side by side, annotating the derating factor, and export the figure.

## Related tools

- **GrOpt** (ADMM-based optimizer that solves the moment-constrained waveform for both the derated logical-axis and rotated physical-axis constraint sets) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used for prototyping and visualizing the gradient waveforms (Jupyter notebook demos in ./python/)) — https://github.com/cmr-group/gropt
- **Matlab** (Alternative wrapper for prototyping and visualizing gradient waveforms via demo_* scripts in ./matlab/) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines into the Python module; built with setup.py build_ext --inplace) — https://github.com/cmr-group/gropt

## Examples

```
cd gropt/python && python setup.py build_ext --inplace && jupyter notebook  # then run the constraint-demo notebook for both derated and rotated constraint sets
```

## Evaluation signals

- Both waveform sets are feasible: each meets the imposed gradient moment constraints within GrOpt's convergence tolerance
- Rotationally invariant waveforms never exceed the derated Gmax/Smax bounds (derating factor annotated on the figure)
- Rotationally variant waveforms reach the full, underated hardware limits, confirming the physical-coordinate solve exploits available headroom
- Per-physical-axis plots confirm the rotated solution respects axis-specific limits for the chosen oblique orientation
- Optimization converges within the expected runtime envelope (acceptable waveforms were reported in 1-40 ms; non-convergence stops at 20,000 iterations)

## Limitations

- The rotationally invariant approach permanently derates hardware limits (by the rotation factor, up to 3), so it cannot exploit full gradient performance for a specific known slice orientation
- Convexity of the solution space with multiple constraints is difficult to prove; feasibility is not guaranteed for every constraint combination
- Choice of N and dt may need adjustment for longer gradient waveforms such as multiecho turbo-spin echo sequences

## Evidence

- [methods] Slice obliquity is handled using either a "rotationally invariant" approach that derates the hardware limitations by 3: [evidence span withheld — anchor status: not_found]
- [methods] or using the "rotationally variant" approach that rotates the relevant constraints and solves for optimized gradient waveforms: [evidence span withheld — anchor status: not_found]
- [readme] This will use Cython to generate the source files for a Python module, and then compile it within the GrOpt folder.: [evidence span withheld — anchor status: external_doc]
