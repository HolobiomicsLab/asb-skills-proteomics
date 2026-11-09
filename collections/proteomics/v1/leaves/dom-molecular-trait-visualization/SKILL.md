---
name: dom-molecular-trait-visualization
description: Use when FT-ICR MS molecular compositional tables with relative abundances
  sampled across treatments (e.g., six experimental warming temperature levels in
  temperate and subtropical regions) must be summarized into per-sample diversity
  indices and tested against the gradient.
when_to_use_negative:
- Input is raw FT-ICR MS spectra that have not yet been converted into a molecular
  compositional table with relative abundances of assigned formulae
- Goal is quantifying deterministic vs stochastic DOM assembly processes (use iDOM's
  commProc) or dark-matter effects on molecular interactions (use iDME) rather than
  diversity-temperature trends
- Goal is DOM-microbe bipartite network specialization (use iDOM's H2) rather than
  univariate molecular diversity indices
edam_operation: http://edamontology.org/operation_3799
tools:
- name: iDOM
  role: R package providing basic diversity functions (richness, Shannon, evenness),
    molecular-trait/CWM routines for RaoQ, dendrogram-based MNTD, and visualization
    of DOM diversity trends
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM diversity computations, per-region linear
    regressions, and plotting are executed
- name: devtools
  role: Used to install the iDOM development version from GitHub before running the
    diversity-temperature workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:6069106614ea8284d3d74474d65d358585ce04a686b95fda3f5e4d656d135ec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dom-molecular-trait-visualization

## Summary

Computes and visualizes DOM molecular diversity indices (richness, Shannon, evenness, RaoQ functional diversity, MNTD) from FT-ICR MS molecular compositional tables across an experimental gradient, using the iDOM R package. Used to test and plot region-specific diversity-temperature relationships such as those in Figure 3A-C.

## When to use

Use when FT-ICR MS molecular compositional tables with relative abundances sampled across treatments (e.g., six experimental warming temperature levels in temperate and subtropical regions) must be summarized into per-sample diversity indices and tested against the gradient.

## When NOT to use

- Input is raw FT-ICR MS spectra that have not yet been converted into a molecular compositional table with relative abundances of assigned formulae
- Goal is quantifying deterministic vs stochastic DOM assembly processes (use iDOM's commProc) or dark-matter effects on molecular interactions (use iDME) rather than diversity-temperature trends
- Goal is DOM-microbe bipartite network specialization (use iDOM's H2) rather than univariate molecular diversity indices

## Inputs

- FT-ICR MS molecular compositional table with relative abundances of assigned molecules per sample
- Sample metadata: experimental treatment levels (six temperature levels) and region labels (temperate vs subtropical)
- Molecular trait assignments for abundance-weighted CWM / RaoQ computation

## Outputs

- Per-sample diversity index values (richness, Shannon, evenness, RaoQ, MNTD) at each temperature level
- Results table (e.g., diversity_temperature_trends.csv) with index values, regression slopes and p-values per region
- Diversity-versus-temperature plots reproducing Figure 3A-C style panels

## How to apply

Install iDOM from GitHub in R via devtools::install_github('jianjunwang/iDOM') and load the bundled experimental warming FT-ICR MS molecular compositional table (relative abundances, six temperature levels, temperate and subtropical regions). Compute per-sample taxonomic-equivalent indices (richness, Shannon, evenness) with iDOM's basic diversity functions, functional diversity as RaoQ via abundance-weighted CWM of molecular traits, and MNTD per assemblage from a molecular dendrogram built on trait-based dissimilarities. Fit linear regressions of each index against temperature level separately per region, extracting slopes and p-values and applying a p < 0.05 significance threshold. Assemble a results table of index values, slopes and p-values per region (e.g., diversity_temperature_trends.csv) and plot the index-versus-temperature trends to reproduce the region-specific patterns of Figure 3A-C.

## Related tools

- **iDOM** (R package providing basic diversity functions (richness, Shannon, evenness), molecular-trait/CWM routines for RaoQ, dendrogram-based MNTD, and visualization of DOM diversity trends) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM diversity computations, per-region linear regressions, and plotting are executed)
- **devtools** (Used to install the iDOM development version from GitHub before running the diversity-temperature workflow)

## Examples

```
install.packages("devtools"); devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Molecular richness decreases significantly with rising temperature in the temperate region (p < 0.05) but shows no significant change in the subtropical region (p > 0.05), matching Figure 3A
- RaoQ and MNTD decrease significantly in the subtropical region (p < 0.05) but not in the temperate region, matching Figure 3B-C
- Results table contains, for each diversity index and region, the index values, regression slope and p-value, and validates against the p < 0.05 threshold
- Diversity indices are computed per sample at each of the six temperature levels, with functional diversity (RaoQ) weighted by relative abundances of assigned molecules

## Limitations

- Requires molecular compositional data with relative abundances and assigned molecular traits; uncharacterized molecules (dark matter) are outside the basic diversity calculations
- Diversity-temperature relationships are region-specific: significance patterns differ between temperate and subtropical regions and must not be pooled across regions
- Basic analyses cover molecular traits, class assignment, and chemical diversity/dissimilarity; advanced questions (assembly processes, dark matter, DOM-microbe networks) need other iDOM functions

## Evidence

- [abstract] we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
- [abstract] The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter).: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [readme] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and dissimilarity.: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [readme] You can install the development version of **iDOM** from [GitHub](https://github.com/) with: devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
