---
name: nmds-pcoa-ordination-of-molecular-composition
description: Use when a sample-by-formula presence/absence or relative-abundance table
  from FT-ICR MS must be visualized in 2D and formally tested for compositional differences
  between predefined sample groups (e.g., subtropical vs temperate regions).
when_to_use_negative:
- The goal is quantifying deterministic vs stochastic DOM assembly processes or dark
  matter effects rather than visualizing/testing compositional dissimilarity — use
  iDOM's commProc or iDME instead
- Input is raw FT-ICR MS peak lists that have not yet been converted to a samples
  x formula presence/absence or relative-abundance compositional table
- The comparison is of molecular traits (H/C, O/C ratios) across Earth systems rather
  than sample-level compositional dissimilarity between predefined groups
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package whose commDis function computes the Jaccard, Bray-Curtis, and UniFrac
    dissimilarity matrices from DOM molecular compositional data
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment for iDOM and for the vegan-based ordination (metaMDS,
    cmdscale) and PERMANOVA (adonis2) steps
- name: vegan
  role: Provides metaMDS/cmdscale for 2D ordination of the commDis dissimilarity matrices
    and adonis2 for permutation-based PERMANOVA testing
- name: devtools
  role: Used to install iDOM from its GitHub repository
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:e91edec128e081ac535300654a04b7441c3a77d5a6afbf8199c72cb0886243f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nmds-pcoa-ordination-of-molecular-composition

## Summary

Project FT-ICR MS DOM molecular compositional dissimilarity (Jaccard, Bray-Curtis, UniFrac from iDOM's commDis) into two-dimensional ordinations with NMDS or PCoA, then test between-group compositional similarity with PERMANOVA (vegan::adonis2).

## When to use

Use when a sample-by-formula presence/absence or relative-abundance table from FT-ICR MS must be visualized in 2D and formally tested for compositional differences between predefined sample groups (e.g., subtropical vs temperate regions).

## When NOT to use

- The goal is quantifying deterministic vs stochastic DOM assembly processes or dark matter effects rather than visualizing/testing compositional dissimilarity — use iDOM's commProc or iDME instead
- Input is raw FT-ICR MS peak lists that have not yet been converted to a samples x formula presence/absence or relative-abundance compositional table
- The comparison is of molecular traits (H/C, O/C ratios) across Earth systems rather than sample-level compositional dissimilarity between predefined groups

## Inputs

- DOM molecular compositional table (samples x molecular formula presence/absence matrix) from FT-ICR MS
- DOM molecular compositional table (samples x molecular formula relative abundance matrix) from FT-ICR MS
- Sample metadata grouping factor (e.g., subtropical vs temperate region labels)

## Outputs

- Pairwise sample-by-sample dissimilarity matrices (Jaccard, Bray-Curtis, UniFrac) from iDOM commDis
- Two-dimensional NMDS or PCoA ordination coordinates colored by climatic region (Figure 3D-style plot)
- PERMANOVA statistics (F, R2, p-value) and dissimilarity/ordination results written to dissimilarity_permanova_results.csv

## How to apply

Install iDOM from GitHub (devtools::install_github("jianjunwang/iDOM")) and load the example DOM molecular compositional table (samples x formula presence/absence and relative abundance). Compute pairwise sample-by-sample dissimilarity matrices with iDOM's commDis: Jaccard on presence/absence data, Bray-Curtis on relative abundances, and UniFrac incorporating molecular trait information. Project each matrix into two dimensions using vegan::metaMDS (NMDS) or vegan::cmdscale (PCoA) and color samples by the grouping factor (climatic region) to reproduce the Figure 3D-style ordination. Run PERMANOVA with vegan::adonis2 using permutations on each dissimilarity matrix against the grouping factor, and report F, R2, and p-value. In the warming case study, the expectation is no significant difference between subtropical and temperate regions (p > 0.05), i.e., similar DOM molecular composition across climate zones.

## Related tools

- **iDOM** (R package whose commDis function computes the Jaccard, Bray-Curtis, and UniFrac dissimilarity matrices from DOM molecular compositional data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment for iDOM and for the vegan-based ordination (metaMDS, cmdscale) and PERMANOVA (adonis2) steps)
- **vegan** (Provides metaMDS/cmdscale for 2D ordination of the commDis dissimilarity matrices and adonis2 for permutation-based PERMANOVA testing)
- **devtools** (Used to install iDOM from its GitHub repository)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); D <- iDOM::commDis(comm); ord <- vegan::metaMDS(D, k = 2); vegan::adonis2(D ~ region, permutations = 999)
```

## Evaluation signals

- PERMANOVA p-value for the subtropical/temperate region factor exceeds 0.05, confirming statistically similar molecular composition between regions
- adonis2 output reports F, R2, and p-value, and all three (matrices, ordination coordinates, statistics) are written to dissimilarity_permanova_results.csv
- Each dissimilarity matrix is square, symmetric, with zero diagonal and matching sample order to the input compositional table
- NMDS ordination reaches acceptable stress and samples colored by climatic region reproduce the reported Figure 3D pattern of overlapping subtropical and temperate samples
- All three metrics (Jaccard on presence/absence, Bray-Curtis on relative abundance, UniFrac with molecular traits) are computed and compared, not just one

## Limitations

- iDOM must be installed from GitHub source (devtools::install_github or install.packages with repos = NULL, type = 'source'), which may require compiling from zip or tar.gz locally
- Results are specific to compositional dissimilarity between defined groups; the case study expectation of no significant region difference (p > 0.05) may not generalize to other datasets
- The package handles FT-ICR MS-derived DOM data types (molecular compositional data, molecular traits, dark matter); it is not intended for generic abundance tables from other instrument types

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and dissimilarity: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [abstract] compositional analyses of chemical diversity and dissimilarity: "the assignment of molecular classes, and the compositional analyses of chemical diversity and dissimilarity"
- [methods] devtools::install_github("jianjunwang/iDOM"): "# install.packages("devtools")
devtools::install_github("jianjunwang/iDOM")"
- [readme] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and dissimilarity.: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
