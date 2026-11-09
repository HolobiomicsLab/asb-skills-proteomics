---
name: permanova-regional-composition-testing
description: Use when FT-ICR MS-derived DOM molecular compositional data (samples
  x molecular formula presence/absence and relative abundance) must be compared between
  predefined groups, e.g., climatic regions, to test compositional similarity.
when_to_use_negative:
- Input is raw FT-ICR MS spectra rather than a processed formula-by-sample compositional
  table (traits/class assignment must be done first with iDOM's basic functions)
- The goal is quantifying DOM assembly processes, dark matter effects, or DOM-microbe
  networks (use commProc, iDME, or H2 instead)
- Groups have unequal dispersion that could confound PERMANOVA significance (differences
  in multivariate spread can mimic compositional differences)
edam_operation: http://edamontology.org/operation_3460
edam_topics: []
tools:
- name: iDOM
  role: R package providing commDis for computing Jaccard, Bray-Curtis, and UniFrac
    dissimilarity matrices from DOM molecular compositional data
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment for iDOM and vegan; installing packages in R from zip
    source is described in the methods
- name: devtools
  role: Used to install iDOM from GitHub via devtools::install_github
- name: vegan
  role: Provides metaMDS/cmdscale for ordination and adonis2 for PERMANOVA on the
    commDis dissimilarity matrices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:ec2c783f3518700caf1dc9d4fd8e244160a30862adad8e1d7326b8788791c338
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  - vegan
  edam_operation: http://edamontology.org/operation_3460
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# permanova-regional-composition-testing

## Summary

Quantifies whether dissolved organic matter (DOM) molecular composition differs between groupings (e.g., subtropical vs temperate regions) by computing sample-by-sample dissimilarity matrices with iDOM's commDis and testing them with PERMANOVA (vegan::adonis2).

## When to use

Use when FT-ICR MS-derived DOM molecular compositional data (samples x molecular formula presence/absence and relative abundance) must be compared between predefined groups, e.g., climatic regions, to test compositional similarity.

## When NOT to use

- Input is raw FT-ICR MS spectra rather than a processed formula-by-sample compositional table (traits/class assignment must be done first with iDOM's basic functions)
- The goal is quantifying DOM assembly processes, dark matter effects, or DOM-microbe networks (use commProc, iDME, or H2 instead)
- Groups have unequal dispersion that could confound PERMANOVA significance (differences in multivariate spread can mimic compositional differences)

## Inputs

- DOM molecular compositional table (samples x molecular formula presence/absence and relative abundance) from FT-ICR MS, e.g., the iDOM experimental warming example dataset
- Sample metadata grouping factor (e.g., subtropical vs temperate climatic region)
- Molecular trait information for UniFrac-based dissimilarity

## Outputs

- Three sample-by-sample dissimilarity matrices (Jaccard, Bray-Curtis, UniFrac) from commDis
- Two-dimensional NMDS or PCoA ordination coordinates colored by climatic region (Figure 3D reproduction)
- PERMANOVA statistics (F, R2, p-value) per dissimilarity matrix, written to dissimilarity_permanova_results.csv

## How to apply

Load the iDOM example DOM molecular compositional table (samples x formula presence/absence and relative abundance) after installing the package from GitHub with devtools::install_github('jianjunwang/iDOM'). Compute pairwise sample-by-sample dissimilarity matrices using iDOM's commDis: Jaccard on presence/absence data, Bray-Curtis on relative-abundance data, and UniFrac incorporating molecular trait information. Optionally project each matrix into two dimensions with NMDS (vegan::metaMDS) or PCoA (vegan::cmdscale) and color samples by the grouping factor to reproduce the Figure 3D ordination. Run PERMANOVA with vegan::adonis2 (with permutations) on each dissimilarity matrix against the grouping factor (e.g., subtropical/temperate region) and record F, R2, and p-value; the case study reports no significant difference between regions (p > 0.05).

## Related tools

- **iDOM** (R package providing commDis for computing Jaccard, Bray-Curtis, and UniFrac dissimilarity matrices from DOM molecular compositional data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment for iDOM and vegan; installing packages in R from zip source is described in the methods)
- **devtools** (Used to install iDOM from GitHub via devtools::install_github)
- **vegan** (Provides metaMDS/cmdscale for ordination and adonis2 for PERMANOVA on the commDis dissimilarity matrices)

## Examples

```
library(iDOM); D_jac <- commDis(pa_table, method="jaccard"); vegan::adonis2(D_jac ~ region, permutations=999)
```

## Evaluation signals

- PERMANOVA p-value for the region factor must exceed 0.05, confirming the reported result that molecular composition is statistically similar between subtropical and temperate regions
- Three dissimilarity matrices (Jaccard, Bray-Curtis, UniFrac) are produced and each is symmetric with zero diagonal (valid distance matrices)
- Ordination (NMDS/PCoA) of samples colored by climatic region reproduces the Figure 3D layout with intermingled subtropical and temperate samples
- Output file dissimilarity_permanova_results.csv contains F, R2, and p-value for each dissimilarity metric

## Limitations

- iDOM is designed for FT-ICR MS-derived DOM data; it handles molecular compositional data, molecular traits, and dark matter, and is not intended for other data types
- commDis supports the three dissimilarity metrics demonstrated (Jaccard, Bray-Curtis, UniFrac); other metrics would need external implementations
- The demonstrated finding is specific to the experimental warming example dataset; other datasets may show significant regional differences

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [readme] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [abstract] We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM"
- [abstract] The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter).: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
