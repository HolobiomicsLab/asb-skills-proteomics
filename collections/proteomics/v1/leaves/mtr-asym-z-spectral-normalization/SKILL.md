---
name: mtr-asym-z-spectral-normalization
description: Use when comparing simulated Z-spectra (from pulseq-cest .seq Bloch-McConnell
  simulation) with measured CEST Z-spectra, or when computing MTR_asym, and raw magnetization
  values must first be placed on a common normalized scale.
when_to_use_negative:
- The dataset lacks an off-resonance reference measurement (no −6 ppm acquisition),
  so the normalization reference point is undefined
- Data are already expressed as normalized Z values or MTR_asym — re-normalizing would
  distort the comparison
- The goal is quantification of pool parameters in vivo, which the article notes is
  'extremely challenging' and out of scope for this Z-spectrum comparison workflow
tools:
- name: pypulseq-cest
  role: Python wrapper whose simulate() function runs the C++ Bloch-McConnell simulation
    and exposes normalize/norm_threshold kwargs to normalize the resulting Z-spectrum
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: pulseq-cest
  role: MATLAB implementation containing the Bloch-McConnell simulation (pulseq-cest-sim)
    used to produce simulated Z-spectra from .seq files
  repo: https://github.com/kherz/pulseq-cest
- name: Pulseq
  role: Defines the .seq file whose 39 saturation offsets (±6 ppm) determine the offset
    grid of the Z-spectrum being normalized
  repo: https://github.com/pulseq/pulseq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:fb383129d23d21b1f34c7863f89a0c9930fcbfe2dddc81a675a9265820278497
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KerstinHut/pypulseq-cest
  tools:
  - C++
  - Pulseq
  - pulseq-cest
  - pypulseq-cest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mtr-asym-z-spectral-normalization

## Summary

Normalizes simulated and measured CEST Z-spectra to the signal at −6 ppm so that Z-spectral curves and MTR_asym curves from pulseq-cest Bloch-McConnell simulations and phantom/scanner measurements can be compared directly on the same scale.

## When to use

Use when comparing simulated Z-spectra (from pulseq-cest .seq Bloch-McConnell simulation) with measured CEST Z-spectra, or when computing MTR_asym, and raw magnetization values must first be placed on a common normalized scale.

## When NOT to use

- The dataset lacks an off-resonance reference measurement (no −6 ppm acquisition), so the normalization reference point is undefined
- Data are already expressed as normalized Z values or MTR_asym — re-normalizing would distort the comparison
- The goal is quantification of pool parameters in vivo, which the article notes is 'extremely challenging' and out of scope for this Z-spectrum comparison workflow

## Inputs

- Simulated Z-spectrum: offsets (39 offsets, ±6 ppm) and water-pool z-magnetization from the pulseq-cest Bloch-McConnell simulator
- Measured Z-spectrum: offsets and mean signal per ROI (e.g. L-arginine / D-glucose phantom tube ROIs)
- ΔB0 map or offset-shift estimates for linear-interpolation correction
- Optional: pulseq-cest yaml config and .seq file passed to the simulate() wrapper

## Outputs

- Normalized Z-spectra (both simulated and measured) with Z(-6 ppm) = 1
- MTR_asym curve computed from the normalized Z-spectra
- Per-ROI comparison table (e.g. z_spectra_comparison.csv) with simulated/measured Z-spectra columns and maximum residuals

## How to apply

Collect the Z-spectrum as a set of saturation offsets (e.g. the 39 offsets spanning ±6 ppm defined in the .seq preparation such as SLExp_3T_Phantom.seq) paired with post-saturation water magnetization values from both the simulation (pulled from the simulator's final magnetization vectors, e.g. via sim.GetFinalMagnetizationVectors() and get_zspec()) and the measurement. Before normalizing, apply ΔB0 correction using linear interpolation between acquired offsets so that offsets align between simulated and measured data. Normalize every Z value — simulated and measured alike — by the signal measured at the −6 ppm offset, which becomes the common reference point of 1.0. Only after this shared normalization compute per-ROI residuals or MTR_asym; in the source phantom study this yielded maximum residuals of 0.009 (ROI 1), 0.008 (ROI 2) and 0.004 (ROI 3). The pypulseq-cest simulate() function exposes normalize (bool) and norm_threshold (value/list/array: threshold for normalization offsets) kwargs to control this step directly.

## Related tools

- **pypulseq-cest** (Python wrapper whose simulate() function runs the C++ Bloch-McConnell simulation and exposes normalize/norm_threshold kwargs to normalize the resulting Z-spectrum) — https://github.com/KerstinHut/pypulseq-cest
- **pulseq-cest** (MATLAB implementation containing the Bloch-McConnell simulation (pulseq-cest-sim) used to produce simulated Z-spectra from .seq files) — https://github.com/kherz/pulseq-cest
- **Pulseq** (Defines the .seq file whose 39 saturation offsets (±6 ppm) determine the offset grid of the Z-spectrum being normalized) — https://github.com/pulseq/pulseq

## Examples

```
sim = simulate(config_file=sim_config, seq_file=seq_file, normalize=True); offsets, mz = get_zspec(m_out=sim.GetFinalMagnetizationVectors(), sp=sim, seq_file=seq_file)
```

## Evaluation signals

- The normalized Z value at −6 ppm equals 1.0 (or is masked out per norm_threshold) for both simulated and measured spectra
- The offset grid matches the .seq definition — 39 offsets spanning ±6 ppm — after ΔB0 linear-interpolation correction
- Per-ROI maximum residuals after normalization are on the order of 0.009 (ROI 1), 0.008 (ROI 2), 0.004 (ROI 3) for the phantom reference case
- get_zspec(m_out, sp, seq_file) returns offset and mz arrays of equal length before and after normalization
- MTR_asym is computed only from the normalized Z-spectrum, not raw magnetization

## Limitations

- Normalization is sensitive to the ΔB0 correction: offsets must first be corrected via linear interpolation between acquired offsets, otherwise the −6 ppm reference is misaligned between simulated and measured data
- The comparison/validation targets (max residuals ~0.009/0.008/0.004) come from a well-controlled L-arginine/D-glucose phantom; such agreement is not expected in vivo
- The normalization behavior of simulate() depends on the norm_threshold setting (value/list/array of offsets), so mismatched thresholds can leave the reference point or exclude wrong offsets

## Evidence

- [methods] Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [readme] These are for example *normalize* (bool: toggle normalization), *norm_threshold* (value/list/array: threshold for normalization offsets), *offsets* (list/array: manually defined x-values): [evidence span withheld — anchor status: not_found]
- [abstract] The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [methods] after a ΔB0 correction using a linear interpolation between acquired offsets: "after a ΔB0 correction using a linear interpolation between acquired offsets"
