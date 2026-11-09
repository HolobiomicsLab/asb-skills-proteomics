---
name: functional-diversity-raoq-cwm-estimation
description: Use when you have an FT-ICR MS molecular compositional table (molecules
  x samples with relative abundances) plus assigned molecular traits, and need a functional
  diversity index per sample to test trait-based responses (e.g., to warming).
when_to_use_negative:
- Input is only a peak list without relative abundances — CWM weighting of traits
  by abundance cannot be computed
- Molecules have no assigned molecular traits or classes, so no trait space exists
  for RaoQ/MNTD computation
- The goal is purely compositional (taxonomic-equivalent) richness/Shannon/evenness
  rather than trait-weighted functional diversity — use iDOM's basic diversity functions
  instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0092
tools:
- name: iDOM
  role: R package providing the molecular-trait routines and diversity functions used
    to compute RaoQ via CWM and MNTD on FT-ICR MS DOM compositional data
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM's molecular-trait and diversity routines
    are executed
- name: devtools
  role: Used to install iDOM from GitHub (devtools::install_github('jianjunwang/iDOM'))
- name: FT-ICR MS
  role: Instrument producing the molecular compositional data (relative abundances
    of DOM molecules) that feed the CWM/RaoQ calculation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:dc5c996ae45f06b7507db65fbab0d07bf3f76f5ab961e45b8558e7e129e1157c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# functional-diversity-raoq-cwm-estimation

## Summary

Estimate the functional diversity of dissolved organic matter (DOM) assemblages by computing Rao's quadratic entropy (RaoQ) through community-weighted means (CWM), weighting molecular traits (e.g., H/C, O/C-derived trait values) by the relative abundances of assigned FT-ICR MS molecules.

## When to use

Use when you have an FT-ICR MS molecular compositional table (molecules x samples with relative abundances) plus assigned molecular traits, and need a functional diversity index per sample to test trait-based responses (e.g., to warming).

## When NOT to use

- Input is only a peak list without relative abundances — CWM weighting of traits by abundance cannot be computed
- Molecules have no assigned molecular traits or classes, so no trait space exists for RaoQ/MNTD computation
- The goal is purely compositional (taxonomic-equivalent) richness/Shannon/evenness rather than trait-weighted functional diversity — use iDOM's basic diversity functions instead

## Inputs

- FT-ICR MS molecular compositional table (molecules x samples) with relative abundances
- Assigned molecular classes / molecular traits (e.g., elemental-ratio-based trait values per molecule)
- Experimental design metadata (e.g., temperature levels, region labels for temperate vs subtropical samples)

## Outputs

- Per-assemblage (per-sample) RaoQ functional diversity values
- Per-assemblage MNTD values from a trait-based molecular dendrogram (optional companion metric)
- Results table (e.g., diversity_temperature_trends.csv) of index values, regression slopes, and p-values per condition/region

## How to apply

Start from the iDOM R package, installed via devtools::install_github('jianjunwang/iDOM'), and load the molecular compositional data (relative abundances of molecules characterized by FT-ICR MS). Compute molecular traits with iDOM's molecular-trait routines, then compute functional diversity as RaoQ via CWM, weighting each molecular trait by the relative abundance of its assigned molecule in each assemblage. Optionally build a molecular dendrogram from trait-based dissimilarities and compute MNTD per assemblage to complement RaoQ with a phylogenetic-equivalent functional metric. Replicate the per-sample index computation across all experimental conditions (e.g., the six temperature levels of the temperate and subtropical warming dataset) so the index can be regressed against the gradient (significance threshold p < 0.05). Assemble index values, slopes, and p-values per region into a results table (e.g., diversity_temperature_trends.csv) for downstream interpretation.

## Related tools

- **iDOM** (R package providing the molecular-trait routines and diversity functions used to compute RaoQ via CWM and MNTD on FT-ICR MS DOM compositional data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM's molecular-trait and diversity routines are executed)
- **devtools** (Used to install iDOM from GitHub (devtools::install_github('jianjunwang/iDOM')))
- **FT-ICR MS** (Instrument producing the molecular compositional data (relative abundances of DOM molecules) that feed the CWM/RaoQ calculation)

## Examples

```
library(devtools); devtools::install_github("jianjunwang/iDOM"); library(iDOM)  # then compute RaoQ via CWM on the bundled warming dataset
```

## Evaluation signals

- RaoQ values are finite and non-negative for every sample/assemblage in the compositional table
- In the warming case study, functional diversity (RaoQ and MNTD) must decrease significantly with rising temperature in the subtropical region (p < 0.05) but not in the temperate region, matching Figure 3A-C
- Regressions of each diversity index against temperature level yield extractable slopes and p-values for both regions, assembled into the expected results-table schema
- CWM computation reproduces abundance weighting: molecules with zero relative abundance contribute zero weight to the trait mean

## Limitations

- Functional diversity estimates depend on the quality/completeness of molecular trait assignment; uncharacterized molecules (dark matter) are excluded from trait weighting
- Trend conclusions are region-specific: RaoQ and MNTD decreased significantly in the subtropical region (p < 0.05) but not in the temperate region, so the metric's temperature response is not universally transferable

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [abstract] The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter).: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [methods] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [abstract] iDOM is an R package providing basic and advanced statistical analyses and visualization for DOM data from FT-ICR MS, handling molecular compositional data, molecular traits: [evidence span withheld — anchor status: not_found]
