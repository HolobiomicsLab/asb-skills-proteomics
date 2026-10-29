---
name: multi-vendor-mri-protocol-transferability-analysis
description: Use when a CEST (or similar) protocol must be reproduced on a scanner
  vendor/site different from the original implementation, and you need to verify whether
  a Pulseq interpreter (e.g., Siemens IDEA SBB vs. Bruker ParaVision) exists and whether
  simulated and measured spectra agree.
when_to_use_negative:
- When the target scanner has no Pulseq interpreter implementation — in the presented
  work the Siemens IDEA SBB is 'so far only available for Siemens scanners', so a
  Bruker (e.g., 14.1 T ParaVision 6) or Philips/Canon/United implementation cannot
  be assumed from this paper
- When only the saturation pulse shapes matter but the readout must also be shared
  — the hybrid design couples the pulseq-defined preparation block with a vendor-native
  readout (e.g., 3D snapshot GRE), so the readout itself is not vendor-independent
- When a quantitative Bloch-McConnell-based parameter fit in vivo is the goal — the
  article states 'Bloch-McConnell based quantification is extremely challenging in
  vivo'
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3444
tools:
- name: pulseq-cest
  role: MATLAB framework providing SequenceSBB loading of .seq-files, the Bloch-McConnell
    simulation (pulseq-cest-sim), and the shared protocol library used to standardize
    CEST preparation periods across sites
  repo: https://github.com/kherz/pulseq-cest
- name: pulseq-cest-library
  role: Repository of approved pre-saturation schemes (.seq) and approved simulation
    settings (yaml sim-library) used as the canonical protocol inputs for cross-site/cross-vendor
    comparison
  repo: https://github.com/kherz/pulseq-cest-library
- name: pypulseq-cest
  role: Python alternative providing the SWIG-wrapped C++ simulator (pySimPulseqSBB),
    seq/yaml handling via BMCTool, and the simulate() function with normalize and
    plot_mtr_asym options for comparing z-spectra
  repo: https://github.com/KerstinHut/pypulseq-cest
- name: Pulseq
  role: Vendor-independent, human-readable open file format in which the CEST preparation
    period is exactly defined and shared across vendors
  repo: https://pulseq.github.io/
- name: MATLAB
  role: Environment for creating pulseq-files, loading and plotting .seq preparation
    blocks, and running the Bloch-McConnell simulation via mex build
- name: C++
  role: Language of the Bloch-McConnell simulation application that loops through
    the pulseq-files and returns magnetization after preparation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:74a281859312485be88036690b6f1223b42660eb4e0fd8549b8c058d8d7c5f56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest
  tools:
  - C++
  - MATLAB
  - Pulseq
  - pulseq-cest
  - pulseq-cest-library
  - pypulseq-cest
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multi-vendor-mri-protocol-transferability-analysis

## Summary

Assesses whether an MRI protocol defined in a vendor-neutral format (Pulseq .seq-files) can actually be transferred and reproduced across scanner vendors and sites, by checking which vendor interpreter implementations exist and comparing simulated versus measured Z-spectra. Grounded in the Pulseq-CEST standard, which aims at multi-site multi-vendor compatibility and reproducibility of CEST experiments.

## When to use

Use when a CEST (or similar) protocol must be reproduced on a scanner vendor/site different from the original implementation, and you need to verify whether a Pulseq interpreter (e.g., Siemens IDEA SBB vs. Bruker ParaVision) exists and whether simulated and measured spectra agree.

## When NOT to use

- When the target scanner has no Pulseq interpreter implementation — in the presented work the Siemens IDEA SBB is 'so far only available for Siemens scanners', so a Bruker (e.g., 14.1 T ParaVision 6) or Philips/Canon/United implementation cannot be assumed from this paper
- When only the saturation pulse shapes matter but the readout must also be shared — the hybrid design couples the pulseq-defined preparation block with a vendor-native readout (e.g., 3D snapshot GRE), so the readout itself is not vendor-independent
- When a quantitative Bloch-McConnell-based parameter fit in vivo is the goal — the article states 'Bloch-McConnell based quantification is extremely challenging in vivo'

## Inputs

- Pulseq .seq-files defining CEST saturation blocks (e.g., APTw_3T_001.seq, APTw_3T_002.seq, APTw_3T_003.seq from the pulseq-cest-library)
- yaml simulation configuration files from the pulseq-cest-library sim-library (pool parameters, e.g., Supporting Information Tables S2–S4)
- Measured Z-spectrum data from the target scanner (e.g., ParaVision 6 / 14.1 T Bruker measurements at deposited acquisition offsets)
- Vendor interpreter availability information (Siemens IDEA hybrid Pulseq-CEST sequence source, available upon request)

## Outputs

- Vendor coverage assessment stating which scanner platforms the interpreter sequence supports (Siemens-only for the presented work)
- Normalized (−6 ppm) simulated vs. measured Z-spectra overlay per protocol and site
- MTRasym(Δω) curves for simulated and measured data at the amide offsets of interest
- Qualitative transferability verdict (good match / mismatch) per protocol and vendor

## How to apply

First, load the consensus pulseq-files (e.g., APTw_3T_001.seq, APTw_3T_002.seq, APTw_3T_003.seq) from the pulseq-cest-library into the SequenceSBB object in MATLAB to inspect the exact CEST preparation period. Next, check vendor availability: the hybrid Pulseq-CEST interpreter sequence in this work was implemented only for the Siemens IDEA framework and tested on three Siemens scanners at three sites; the paper explicitly notes the Pulseq SBB is 'so far only available for Siemens scanners' and that interpreter designs for Philips, United or Canon are still needed. Then, for any vendor where measurement data exist, run the C++ Bloch-McConnell simulation (SimPulseqSBB via mex or the SWIG-wrapped pySimPulseqSBB in pypulseq-cest) using the yaml sim-library settings, apply ΔB0 correction by linear interpolation between acquired offsets, normalize both simulated and measured Z-spectra by the measurement at −6 ppm, and compute MTRasym = (S(−Δω) − S(Δω))/S0. Judge transferability by the paper's qualitative criterion: the exact pulseq-defined preparation period combined with the flexible simulation should yield a good match between simulations and measurements (no numeric threshold is reported).

## Related tools

- **pulseq-cest** (MATLAB framework providing SequenceSBB loading of .seq-files, the Bloch-McConnell simulation (pulseq-cest-sim), and the shared protocol library used to standardize CEST preparation periods across sites) — https://github.com/kherz/pulseq-cest
- **pulseq-cest-library** (Repository of approved pre-saturation schemes (.seq) and approved simulation settings (yaml sim-library) used as the canonical protocol inputs for cross-site/cross-vendor comparison) — https://github.com/kherz/pulseq-cest-library
- **pypulseq-cest** (Python alternative providing the SWIG-wrapped C++ simulator (pySimPulseqSBB), seq/yaml handling via BMCTool, and the simulate() function with normalize and plot_mtr_asym options for comparing z-spectra) — https://github.com/KerstinHut/pypulseq-cest
- **Pulseq** (Vendor-independent, human-readable open file format in which the CEST preparation period is exactly defined and shared across vendors) — https://pulseq.github.io/
- **MATLAB** (Environment for creating pulseq-files, loading and plotting .seq preparation blocks, and running the Bloch-McConnell simulation via mex build)
- **C++** (Language of the Bloch-McConnell simulation application that loops through the pulseq-files and returns magnetization after preparation)

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='config.yaml', seq_file='APTw_3T_001.seq', show_plot=True, normalize=True, plot_mtr_asym=True)
```

## Evaluation signals

- Vendor audit result matches the article: the hybrid Pulseq-CEST interpreter was implemented only for the Siemens IDEA framework and tested on three Siemens scanners at three sites with two scanner models and different software baselines; no Bruker implementation is reported
- Both simulated and measured Z-spectra are normalized by the measurement at −6 ppm before comparison
- ΔB0 correction was applied via linear interpolation between acquired offsets on measured data
- The comparison shows a good (qualitative) match between simulated and measured Z-spectra/contrast for the consensus APTw protocols, per the paper's acceptance criterion
- The .seq-files used are exactly the library consensus protocols (APTw_3T_001/002/003) and the yaml settings come from the sim-library, ensuring the same pulseq-files are simulated as are played out on the scanner

## Limitations

- The Pulseq SBB (interpreter) is so far only available for Siemens scanners; no Bruker, Philips, United, or Canon implementation is reported in the presented results
- Source code for the Siemens IDEA interpreter sequence is not publicly available — only upon request
- Interpreter software for other manufacturers (Philips, United, Canon) is needed for universal, truly vendor-independent application
- The presented CEST-MRF demonstration is a work-in-progress method undergoing active development, and comparison with previous CEST-MRF studies was beyond the scope of the work
- Bloch-McConnell-based quantification is extremely challenging in vivo, limiting quantitative (as opposed to qualitative) transferability claims
- Acceptance of simulated-vs-measured agreement is qualitative ('good match'); no numeric threshold is reported in the paper

## Evidence

- [discussion] Siemens-only interpreter availability: "While the SBB is so far only available for Siemens scanners"
- [intro] Siemens IDEA implementation and testing scope: "we implemented such a hybrid Pulseq-CEST sequence for the Siemens IDEA (Integrated Development Environment for Applications, Siemens Healthineers, Erlangen, Germany) framework and"
- [intro] vendor-independence rationale: "Using Pulseq for the CEST preparation part in the sequence theoretically enables a vendor-independent approach, provided that a Pulseq interpreter sequence is available for each vendor."
- [methods] normalization and ΔB0 filter steps: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [abstract] qualitative acceptance criterion: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [discussion] interpreter needed for other manufacturers: "A design of the interpreter software for other manufacturers (e.g. Philips, United or Canon) is needed for a universal application."
