---
name: admm-constrained-optimization-execution
description: Use when a time-optimal MRI gradient waveform must be generated on-the-fly
  from a flexible set of input constraints (moments, eddy currents, PNS, b-value,
  duty cycle) using GrOpt's ADMM solver.
when_to_use_negative:
- The problem requires optimizing readout trajectories or slice-select gradients,
  which the GrOpt toolbox has not been used to optimize
- A proof of convexity is required for the multi-constraint solution space, which
  is difficult to establish
- Concomitant field terms, acoustic noise, or gradient heating limits must be constrained
  — these are future work and not yet included in the toolbox
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3300
- http://edamontology.org/topic_3444
tools:
- name: GrOpt
  role: ADMM-based constrained optimizer that takes input constraints and seeks a
    feasible time-optimal gradient waveform
  repo: https://github.com/cmr-group/gropt
- name: C
  role: Language of the core optimization kernels, written for maximum compatibility
    on multiple scanner systems
  repo: https://github.com/cmr-group/gropt
- name: Cython
  role: Wraps the C optimization routines to build the importable Python module
  repo: https://github.com/cmr-group/gropt
- name: Python
  role: Wrapper used for prototyping and visualizing gradient waveforms via Jupyter
    notebook demos
  repo: https://github.com/cmr-group/gropt
- name: Matlab
  role: Wrapper compiled via make.m with mex, used for prototyping and visualization
  repo: https://github.com/cmr-group/gropt
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  - task_008
  source_papers:
  - doi: 10.1002/mrm.28384
    title: A gradient optimization toolbox for general purpose time‐optimal MRI gradient
      waveform design
  source_content_hash: sha256:121c68a2cbcb4ee2aa9166e293908c7ab68c9e91c15beac28af11542752dd6b1
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3300
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# admm-constrained-optimization-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute GrOpt's ADMM-based constrained optimizer (C kernels wrapped in Cython/Python or Matlab) to solve MRI gradient waveform design problems defined by hardware limits (Gmax, Smax), raster time dt, and modular constraints such as gradient moment nulling, eddy-current nulling, and PNS limits.

## When to use

Use when a time-optimal MRI gradient waveform must be generated on-the-fly from a flexible set of input constraints (moments, eddy currents, PNS, b-value, duty cycle) using GrOpt's ADMM solver.

## When NOT to use

- The problem requires optimizing readout trajectories or slice-select gradients, which the GrOpt toolbox has not been used to optimize
- A proof of convexity is required for the multi-constraint solution space, which is difficult to establish
- Concomitant field terms, acoustic noise, or gradient heating limits must be constrained — these are future work and not yet included in the toolbox

## Inputs

- Gradient design problem specification: Gmax, Smax, raster time dt, waveform length N
- Modular constraint set (e.g., moment nulling M0/M1/M2, eddy-current nulling, PNS_thresh, b-value target, duty-cycle target)
- Hardware limits for a specific scanner platform (e.g., GE MR750/EPIC DV26.0R01, Siemens Skyra/IDEA VE11C)

## Outputs

- Feasible/time-optimal gradient waveform(s) (G_out, N_out) with duration T = N*dt
- Constraint-satisfaction residuals (gradient moments, achieved b-value)
- Compute time per optimization call (typically 1-40 ms)

## How to apply

Build the C optimization kernels and expose them via the Cython wrapper by running `python setup.py build_ext --inplace` in the python/ directory (or compile the Matlab wrappers via make.m). Formulate the design problem by specifying Gmax, Smax, gradient raster time dt, waveform length N, and stacking modular constraints (moment nulling M0/M1/M2, eddy-current time-constant suppression, PNS via Schulte or Hebrank models, b-value maximization, or duty-cycle minimization). Run the optimization, which seeks a feasible gradient waveform subject to the constraints within a specified tolerance; the solver is stopped after 20,000 iterations if no feasible solution is found. For time-optimal designs, run the outer search over waveform durations T in a multi-threaded fashion, warm-starting target-dt final optimization from higher-dt solutions. Judge success by checking that the resulting waveform satisfies moment and hardware constraints within tolerance and that per-waveform compute times fall within the reported 1-40 ms range.

## Related tools

- **GrOpt** (ADMM-based constrained optimizer that takes input constraints and seeks a feasible time-optimal gradient waveform) — https://github.com/cmr-group/gropt
- **C** (Language of the core optimization kernels, written for maximum compatibility on multiple scanner systems) — https://github.com/cmr-group/gropt
- **Cython** (Wraps the C optimization routines to build the importable Python module) — https://github.com/cmr-group/gropt
- **Python** (Wrapper used for prototyping and visualizing gradient waveforms via Jupyter notebook demos) — https://github.com/cmr-group/gropt
- **Matlab** (Wrapper compiled via make.m with mex, used for prototyping and visualization) — https://github.com/cmr-group/gropt

## Examples

```
cd python && python setup.py build_ext --inplace && python -c "import gropt; G_out, N_out = gropt.optimize(gmax=0.05, smax=100, dt=10e-6, moment_params=[[0,0,0,0,0],[0,1,0,0,0]], verbose=0)"
```

## Evaluation signals

- Each optimized waveform satisfies its moment (M0/M1/M2) and hardware (Gmax, Smax) constraints within the specified residual tolerance
- Per-waveform compute times fall within the paper's reported 1-40 ms range for acceptable waveform generation
- The optimization terminates with a feasible solution within 20,000 iterations (stopping earlier without feasibility signals failure)
- Waveforms generated with the same constraints on different vendor platforms (GE vs Siemens) are functionally equivalent in moments, b-value, and duration
- Fitted compute-time scaling exponents are consistent with the paper's reported O(n^0.90) to O(n^1.32) range

## Limitations

- If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations
- With multiple constraints, the convexity of the solution space is difficult to prove; randomized-start tests are not a complete proof
- Whether a waveform is truly time-optimal is difficult to answer; brute-force verification is computationally very expensive
- The choice of gradient size (N) and raster time (dt) may need to be adjusted for longer gradient waveforms such as multiecho turbo-spin echo sequences
- Concomitant field terms, acoustic noise, and gradient heating limits are not yet available as constraint options

## Evidence

- [methods] The GrOpt routine is based on the alternating direction method of multipliers (ADMM), with modifications made to handle any number of constraints: [evidence span withheld — anchor status: not_found]
- [methods] If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations: "If no feasible solution exists, or cannot be found by the optimization, the process is stopped after 20,000 iterations"
- [abstract] it was observed that acceptable gradient waveforms could be generated in 1-40 ms.: "it was observed that acceptable gradient waveforms could be generated in 1-40 ms."
- [methods] searching different temporal durations (T) in a multi‐threaded fashion and performing the initial search of T with higher dt to solve quickly, and using those high‐dt waveforms to warm start the: "searching different temporal durations (T) in a multi‐threaded fashion and performing the initial search of T with higher dt to solve quickly, and using those high‐dt waveforms to warm start the"
- [readme] This will use Cython to generate the source files for a Python module, and then compile it within the GrOpt folder.: [evidence span withheld — anchor status: external_doc]
