---
name: magnetic-resonance-fingerprinting-dictionary-matching
description: Use when quantitative CEST parameter maps (fMT, kMT, fAmide, kAmide)
  must be derived from spin-lock-based CEST-MRF acquisitions, especially for cross-site
  or multi-scanner reproducibility studies.
when_to_use_negative:
- When the goal is only qualitative APT-weighted imaging or Z-spectrum comparison
  without quantitative exchange-parameter estimation
- When the scanner vendor does not support the Pulseq sequence building block (SBB
  so far only available for Siemens scanners; interpreter software for Philips, United
  or Canon is needed)
- When in vivo Bloch-McConnell-based absolute quantification is required with strict
  accuracy claims — the article notes such quantification is extremely challenging
  in vivo
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: Pulseq-CEST
  role: Provides the standard .seq-file definition of the CEST-MRF preparation periods
    and the pulseq-cest-library of approved pre-saturation schemes used identically
    for measurement and simulation
  repo: https://github.com/kherz/pulseq-cest-library
- name: Pulseq
  role: Open file format and MATLAB/C++ framework in which the spin-lock saturation
    schedules and interpreter sequences are defined and read
  repo: https://github.com/pulseq/pulseq
- name: C++ Bloch-McConnell simulation (SimPulseqSBB)
  role: Native C++ application/mex-function that loops through the pulseq-files and
    performs Bloch-McConnell simulations to build the amide and MTC dictionaries
  repo: https://github.com/kherz/pulseq-cest
- name: pypulseq-cest
  role: Python implementation exposing the C++ simulation (pySimPulseqSBB) via a simulate()
    function with config (yaml) and seq-file inputs, with normalization options
  repo: https://github.com/KerstinKaspar/pypulseq-cest
- name: MATLAB
  role: Environment for sequence definition, running the mex-function simulations,
    SequenceSBB verification, and dictionary matching
- name: yamlmatlab
  role: MATLAB YAML reader used to load sim-library simulation configuration files
    alongside the .seq files
  repo: https://github.com/ewiger/yamlmatlab
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mrm.28825
    title: 'Pulseq‐CEST: Towards multi‐site multi‐vendor compatibility and reproducibility
      of CEST experiments using an open‐source sequence standard'
  source_content_hash: sha256:a3f1b46b9978ed1b0e74cb5f273706a2ba4ae75e947b79437ebcbbb6e55becf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kherz/pulseq-cest-library
  tools:
  - C++
  - C++ Bloch-McConnell simulation (SimPulseqSBB)
  - MATLAB
  - Pulseq
  - Pulseq-CEST
  - pypulseq-cest
  - yamlmatlab
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Magnetic resonance fingerprinting dictionary matching

## Summary

Generates Bloch-McConnell signal dictionaries from spin-lock-based CEST-MRF pulseq sequences (amide-weighted and semi-solid MTC schedules) and matches measured voxel-wise Z-spectra against them to compute quantitative exchange parameter maps (fMT, kMT, fAmide, kAmide) across multiple MR sites.

## When to use

Use when quantitative CEST parameter maps (fMT, kMT, fAmide, kAmide) must be derived from spin-lock-based CEST-MRF acquisitions, especially for cross-site or multi-scanner reproducibility studies.

## When NOT to use

- When the goal is only qualitative APT-weighted imaging or Z-spectrum comparison without quantitative exchange-parameter estimation
- When the scanner vendor does not support the Pulseq sequence building block (SBB so far only available for Siemens scanners; interpreter software for Philips, United or Canon is needed)
- When in vivo Bloch-McConnell-based absolute quantification is required with strict accuracy claims — the article notes such quantification is extremely challenging in vivo

## Inputs

- Pulseq .seq files for the CEST-MRF preparation blocks (MRF_Amide.seq, MRF_MT.seq, with T1prep.seq, T2prep.seq and WASABI.seq)
- 31-scan spin-lock schedule definitions (B1 0–4 μT at 3.5 ppm for amide; B1 0.2–4 μT at offsets 6–14 ppm for MTC)
- Multisite CEST-MRF image acquisitions (EPI readout) from healthy volunteers
- WASABI-derived B0 and B1 inhomogeneity correction maps
- T1 maps and gray/white matter segmentation masks (SPM)

## Outputs

- Bloch-McConnell simulation dictionaries (amide-weighted and semi-solid MTC)
- Voxel-wise quantitative parameter maps: fMT, kMT, fAmide, kAmide per site
- Cross-site quantitative comparison of the four parameter maps

## How to apply

First verify with SequenceSBB that the .seq files (MRF_Amide.seq and MRF_MT.seq, plus T1prep.seq, T2prep.seq and WASABI.seq) from the pulseq-cest supporting material use the same preparation-period definition for measurement and simulation. Generate two dictionaries with the native C++ Bloch-McConnell simulation: an amide-weighted dictionary from the 31-scan spin-lock schedule with B1 varying 0–4 μT at a constant 3.5 ppm offset, and a semi-solid MTC dictionary from the same 31-scan schedule with B1 0.2–4 μT at offsets 6–14 ppm modeling the MTC pool with a Super-Lorentzian line shape. Correct the measured data for B0 and B1 inhomogeneities (WASABI-derived corrections) and normalize Z-spectra (e.g. by the measurement at −6 ppm) before matching. Perform dictionary-based signal matching according to Perlman et al. on motion-corrected, segmented (gray/white matter masks via SPM) images to produce the four parameter maps per site. Judge correctness by visual and quantitative coherence of fMT, kMT, fAmide and kAmide maps across all sites (Tuebingen Prisma, Boston Prisma, Erlangen Trio), matching the multisite consistency reported in the paper.

## Related tools

- **Pulseq-CEST** (Provides the standard .seq-file definition of the CEST-MRF preparation periods and the pulseq-cest-library of approved pre-saturation schemes used identically for measurement and simulation) — https://github.com/kherz/pulseq-cest-library
- **Pulseq** (Open file format and MATLAB/C++ framework in which the spin-lock saturation schedules and interpreter sequences are defined and read) — https://github.com/pulseq/pulseq
- **C++ Bloch-McConnell simulation (SimPulseqSBB)** (Native C++ application/mex-function that loops through the pulseq-files and performs Bloch-McConnell simulations to build the amide and MTC dictionaries) — https://github.com/kherz/pulseq-cest
- **pypulseq-cest** (Python implementation exposing the C++ simulation (pySimPulseqSBB) via a simulate() function with config (yaml) and seq-file inputs, with normalization options) — https://github.com/KerstinKaspar/pypulseq-cest
- **MATLAB** (Environment for sequence definition, running the mex-function simulations, SequenceSBB verification, and dictionary matching)
- **yamlmatlab** (MATLAB YAML reader used to load sim-library simulation configuration files alongside the .seq files) — https://github.com/ewiger/yamlmatlab

## Examples

```
from pypulseq_cest.simulate import simulate; sim = simulate(config_file='config_example.yaml', seq_file='MRF_Amide.seq', normalize=True)
```

## Evaluation signals

- SequenceSBB confirms the preparation-period definition in the .seq file is identical for measurement and simulation
- Simulated and measured Z-spectra show a good match after exact definition of the CEST preparation period (paper's abstract finding)
- All four parameter maps (fMT, kMT, fAmide, kAmide) are produced for every site (Tuebingen Prisma, Boston Prisma, Erlangen Trio)
- Cross-site fMT, kMT, fAmide and kAmide maps are visually and quantitatively coherent, reproducing the multisite consistency shown in the paper
- Z-spectra normalized consistently (e.g. by the measurement at −6 ppm) before dictionary matching

## Limitations

- The Pulseq SBB is so far only available for Siemens scanners; interpreter software for other manufacturers (e.g. Philips, United or Canon) is needed for universal application
- Source code for the Siemens IDEA interpreter sequence is not publicly available; only available upon request
- The presented CEST-MRF method represents a work-in-progress state undergoing active development
- Bloch-McConnell-based quantification is extremely challenging in vivo
- Comparison with previous CEST-MRF studies was not performed; it is beyond the scope of the presented work

## Evidence

- [methods] spin-lock schedule dictionary definition: "an application was written in C++ that loops through the pulseq-files, performs Bloch-McConnell simulations for the respective sequence events"
- [abstract] multisite coherence finding: "we could show coherent multisite results for a sophisticated CEST method, highlighting the benefits regarding protocol sharing and reproducibility"
- [abstract] simulation-measurement match: "The exact definition of the CEST preparation period, in combination with the flexible simulation, leads to a good match between simulations and measurements"
- [methods] Z-spectrum normalization convention: "Both simulated and measured Z-spectra were normalized by the measurement at −6 ppm"
- [readme] python simulation entry point: [evidence span withheld — anchor status: not_found]
- [discussion] SBB vendor limitation: "While the SBB is so far only available for Siemens scanners"
