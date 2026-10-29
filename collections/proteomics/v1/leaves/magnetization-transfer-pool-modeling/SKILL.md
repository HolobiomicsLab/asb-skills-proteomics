---
name: magnetization-transfer-pool-modeling
description: Use when the goal is to simulate, compare, or optimize CEST pre-saturation
  protocols defined as Pulseq .seq-files — e.g., predicting the Z-spectrum of an APTw
  preparation block before scanner time, or checking that a shared protocol behaves
  as expected.
when_to_use_negative:
- The input is raw scanner k-space or image data rather than a Pulseq .seq-file —
  this skill simulates the preparation block only, not image reconstruction or post-processing.
- You need in vivo Bloch-McConnell-based quantification — the article notes this is
  'extremely challenging in vivo'; the simulation is for protocol testing/optimization,
  not quantitative pool-size fitting of patient data.
- The sequence of interest cannot be expressed as a Pulseq saturation block (e.g.,
  vendor-native sequences without a Pulseq interpreter).
edam_topics:
- http://edamontology.org/topic_3384
tools:
- name: pulseq-cest-sim
  role: Bloch-McConnell simulation core that reads .seq-files and simulates the saturation
    events per offset
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python parser and simulate() entry point wrapping the C++ solver (pySimPulseqSBB
    via SWIG)
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: pulseq-cest-library
  role: Source of approved .seq saturation schemes and yaml simulation settings (Lorentzian/Super-Lorentzian
    3 T configs)
  repo: https://github.com/kherz/pulseq-cest-library
- name: Pulseq
  role: Open file format and MATLAB/C++ readers defining the .seq-file structure consumed
    by the simulator
  repo: https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- name: yamlmatlab
  role: MATLAB yaml reader used to load simulation configuration files
  repo: https://github.com/ewiger/yamlmatlab
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:81c8f759cfc123dd3667c703007ecc009984b375b0c61fbc898aa0e29ea6c800
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - Pulseq
  - pulseq-cest-library
  - pulseq-cest-sim
  - pypulseq-cest
  - yamlmatlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# magnetization-transfer-pool-modeling

## Summary

Configure and run a Bloch-McConnell magnetization-exchange simulation of CEST saturation blocks defined in Pulseq .seq-files, modeling one or more CEST pools plus a semi-solid MTC pool to produce a Z-spectrum in silico. This enables testing and optimization of CEST preparation periods using the exact same .seq file that is played out on the scanner.

## When to use

Use when the goal is to simulate, compare, or optimize CEST pre-saturation protocols defined as Pulseq .seq-files — e.g., predicting the Z-spectrum of an APTw preparation block before scanner time, or checking that a shared protocol behaves as expected.

## When NOT to use

- The input is raw scanner k-space or image data rather than a Pulseq .seq-file — this skill simulates the preparation block only, not image reconstruction or post-processing.
- You need in vivo Bloch-McConnell-based quantification — the article notes this is 'extremely challenging in vivo'; the simulation is for protocol testing/optimization, not quantitative pool-size fitting of patient data.
- The sequence of interest cannot be expressed as a Pulseq saturation block (e.g., vendor-native sequences without a Pulseq interpreter).

## Inputs

- Pulseq .seq-file defining the CEST saturation preparation block (e.g., seq_example.seq, APTw_3T_example.seq)
- yaml simulation configuration file with pool parameters (e.g., config_example.yaml from the sim-library)
- Pulseq-cest-library seq-library entries with approved pre-saturation schemes (MATLAB and Python compatible)

## Outputs

- Z-spectrum (per-offset water-pool Z-magnetization values), e.g., simulated_z_spectrum.csv with one Z value per defined saturation offset
- Final magnetization vectors of the water pool retrievable via sim.GetFinalMagnetizationVectors()
- Optional MTR_asym plot when plot_mtr_asym is toggled in the Python simulate() call

## How to apply

Load the .seq-file (e.g., examples/APTw_3T_example.seq) with SequenceSBB.read() in MATLAB or via the pypulseq-cest parser, optionally inspecting the saturation events with seq.plotSaturationPhase(). Configure the tissue model with one or more CEST pools plus a semi-solid MTC pool, choosing a Lorentzian or Super-Lorentzian line shape for the semi-solid pool (standard 3 T settings ship as approved yaml configs in the pulseq-cest-library sim-library). The C++ Bloch-McConnell solver (compiled as a MATLAB mex or wrapped with SWIG for Python via pySimPulseqSBB) loops through the sequence events of the pulseq-file and returns the magnetization vector after the preparation period for each saturation offset. Assemble the per-offset Z values into a Z-spectrum and normalize by the measurement/simulation at −6 ppm, which is the convention used to compare simulated and measured data in the paper. Judgment of correctness: simulateExample()/sim_example() runs without error, one Z value exists per defined offset, and normalized Z values fall in the 0–1 range.

## Related tools

- **pulseq-cest-sim** (Bloch-McConnell simulation core that reads .seq-files and simulates the saturation events per offset) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python parser and simulate() entry point wrapping the C++ solver (pySimPulseqSBB via SWIG)) — https://github.com/KerstinHut/pypulseq-cest
- **pulseq-cest-library** (Source of approved .seq saturation schemes and yaml simulation settings (Lorentzian/Super-Lorentzian 3 T configs)) — https://github.com/kherz/pulseq-cest-library
- **Pulseq** (Open file format and MATLAB/C++ readers defining the .seq-file structure consumed by the simulator) — https://github.com/pulseq/pulseq/releases/tag/v1.3.1
- **yamlmatlab** (MATLAB yaml reader used to load simulation configuration files) — https://github.com/ewiger/yamlmatlab

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='pypulseq_cest/example_library/config_example.yaml', seq_file='pypulseq_cest/example_library/seq_example.seq', show_plot=True, normalize=True)
```

## Evaluation signals

- simulateExample() (MATLAB) or sim_example() (Python) executes to completion without error
- Output file (simulated_z_spectrum.csv) exists and contains exactly one Z value per saturation offset defined in the .seq-file
- All normalized Z values fall in the range 0–1 after normalization by the −6 ppm offset
- Simulated and measured Z-spectra normalized at −6 ppm show a good match, per the article's validation criterion
- MATLAB and Python paths give identical inputs/outputs when fed the same .seq and yaml configuration

## Limitations

- Bloch-McConnell based quantification is extremely challenging in vivo; the simulation supports protocol testing/optimization rather than direct quantitative pool mapping.
- The C++ core must be compiled (mex with Microsoft Visual C++ 2017 on 64-bit Windows 10) or installed via pre-built distributions for Python 3.6–3.9 on Linux/Windows.
- The yaml config's line-shape choice (Lorentzian vs. Super-Lorentzian) and pool parameters strongly determine results; using unapproved, non-library settings requires careful validation.

## Evidence

- [methods] C++ solver loops through seq events: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [methods] simulation returns magnetization after preparation: "performs Bloch-McConnell simulations for the respective sequence events37, and returns the current magnetization vector after preparation"
- [methods] -6 ppm normalization convention: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [readme] simulate and compare .seq-files: [evidence span withheld — anchor status: not_found]
