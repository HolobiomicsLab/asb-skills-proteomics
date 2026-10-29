---
name: z-spectrum-normalization-and-mtrasym-calculation
description: Use when you have simulated and/or measured CEST Z-spectra (water z-magnetization
  vs saturation offset) and need a common normalized scale and asymmetric-contrast
  (MTRasym) curves before comparing protocols or against scanner data.
when_to_use_negative:
- When the Z-spectra have not been B0-corrected and large field inhomogeneity shifts
  would corrupt the linear-interpolation step — correct ΔB0 first
- When the goal is full Bloch-McConnell-based quantification of pool parameters rather
  than normalized contrast curves, since the paper notes Bloch-McConnell based quantification
  is extremely challenging in vivo
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3306
- http://edamontology.org/topic_3399
tools:
- name: Pulseq-CEST
  role: Provides the C++ Bloch-McConnell simulation whose output Z-spectra are normalized
    and compared with measurements
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python simulate() function with normalize/norm_threshold/plot_mtr_asym kwargs
    and get_zspec for extracting offsets and z-magnetization
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: MATLAB
  role: Used to load .seq files into the SequenceSBB object and to run/install the
    pulseq-cest MATLAB stack for simulation inputs
- name: Python
  role: Environment for the pypulseq-cest/ BMCTool post-processing and plotting of
    normalized Z-spectra and MTRasym
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:f99bf26b1c2790f8691d74f80b184ab811783e5086870325ce233f9cebf6643f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - MATLAB
  - Pulseq-CEST
  - Python
  - pypulseq-cest
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3306
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Z-spectrum normalization and MTRasym calculation

## Summary

Post-processing of CEST Z-spectra that applies ΔB0 correction via linear interpolation between acquired offsets, normalizes both simulated and measured Z-spectra by the signal at −6 ppm, and computes MTRasym(Δω) = (S(−Δω) − S(Δω))/S0 so that Bloch-McConnell simulations of pulseq-files can be overlaid against scanner measurements (e.g., 14.1 T Bruker ParaVision 6 data).

## When to use

Use when you have simulated and/or measured CEST Z-spectra (water z-magnetization vs saturation offset) and need a common normalized scale and asymmetric-contrast (MTRasym) curves before comparing protocols or against scanner data.

## When NOT to use

- When the Z-spectra have not been B0-corrected and large field inhomogeneity shifts would corrupt the linear-interpolation step — correct ΔB0 first
- When the goal is full Bloch-McConnell-based quantification of pool parameters rather than normalized contrast curves, since the paper notes Bloch-McConnell based quantification is extremely challenging in vivo

## Inputs

- Measured Z-spectrum data (e.g., 14.1 T Bruker ParaVision 6 Z-spectrum measurements at deposited acquisition offsets)
- Simulated Z-spectra from the C++ Bloch-McConnell simulation (SimPulseqSBB / pySimPulseqSBB) run on APTw .seq files (APTw_3T_001.seq, APTw_3T_002.seq, APTw_3T_003.seq)
- SimulationParameters object or magnetization vectors from pypulseq-cest simulate()
- Offset values (ppm) at which saturation was acquired/simulated

## Outputs

- Normalized Z-spectra (both simulated and measured, normalized at −6 ppm)
- MTRasym(Δω) = (S(−Δω) − S(Δω))/S0 curves per voxel or ROI
- Overlay comparison plots of simulated vs. measured normalized Z-spectra and MTRasym (e.g., bruker_zspectra_comparison.png)

## How to apply

First apply ΔB0 correction to the measured Z-spectrum using linear interpolation between the acquired offsets, as described in the paper's methods. Then normalize both simulated and measured Z-spectra by the measurement at −6 ppm so the curves share a common reference scale. Next compute MTRasym(Δω) = (S(−Δω) − S(Δω))/S0 voxel- or ROI-wise, focusing on the amide offsets of interest for the APTw protocols. Finally overlay the normalized Z-spectra and MTRasym values from the C++ Bloch-McConnell simulation (run on the .seq files) with the measured data to judge agreement; the paper's acceptance criterion is a good qualitative match between simulated and measured Z-spectra/contrast. In the pypulseq-cest Python stack, the simulate function exposes normalize (toggle normalization) and norm_threshold (normalization offsets) keyword arguments, and returns a SimulationParameters object whose final magnetization vectors can be converted to offsets/z-magnetization via get_zspec.

## Related tools

- **Pulseq-CEST** (Provides the C++ Bloch-McConnell simulation whose output Z-spectra are normalized and compared with measurements) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python simulate() function with normalize/norm_threshold/plot_mtr_asym kwargs and get_zspec for extracting offsets and z-magnetization) — https://github.com/KerstinHut/pypulseq-cest
- **MATLAB** (Used to load .seq files into the SequenceSBB object and to run/install the pulseq-cest MATLAB stack for simulation inputs)
- **Python** (Environment for the pypulseq-cest/ BMCTool post-processing and plotting of normalized Z-spectra and MTRasym)

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='config.yaml', seq_file='APTw_3T_001.seq', show_plot=True, normalize=True, plot_mtr_asym=True)
```

## Evaluation signals

- Both simulated and measured Z-spectra normalized by the value at −6 ppm should be on a comparable scale before overlay
- MTRasym values at amide offsets should show a good qualitative match between simulated and measured data (paper's acceptance criterion: no numeric threshold reported)
- ΔB0 correction via linear interpolation between acquired offsets should be verified as applied to measured data before asymmetry computation
- Output comparison figure exists, is a valid PNG, and contains overlays for all three APTw protocols (APTw_3T_001–003.seq)
- Retrieved offsets and z-magnetization from get_zspec(m_out, sp, seq_file) should span the deposited acquisition offsets used in the simulation

## Limitations

- The paper's acceptance is qualitative ('a good match between simulations and measurements'); no numeric agreement threshold is reported
- ΔB0 correction is performed by linear interpolation between acquired offsets, which assumes sufficient offset sampling around each point
- Bloch-McConnell based quantification is extremely challenging in vivo, so normalized Z-spectra/MTRasym are contrast-level, not quantitative pool-parameter, outputs

## Evidence

- [methods] Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [methods] ΔB0 correction via linear interpolation between acquired offsets: "after a ΔB0 correction using a linear interpolation between acquired offsets"
- [abstract] The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [readme] The simulate function requires a *config_file* and *seq_file* argument. The *show_plot* argument (default=False) allows to toggle a plotting functionality.: [evidence span withheld — anchor status: not_found]
- [discussion] since Bloch-Mc McConnell based quantification is extremely challenging in vivo: "since Bloch-McConnell based quantification is extremely challenging in vivo"
