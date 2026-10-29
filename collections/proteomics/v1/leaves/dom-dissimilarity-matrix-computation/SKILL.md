---
name: dom-dissimilarity-matrix-computation
description: Use when you have a sample-by-molecular-formula DOM table (presence/absence
  plus relative abundance) from FT-ICR MS and need chemical diversity/dissimilarity
  metrics or a test of whether DOM composition differs between sample groupings.
when_to_use_negative:
- Input is not FT-ICR MS-derived molecular compositional data (e.g., you have 16S
  amplicon ASV tables or bulk optical DOM indices like EEM/PARAFAC components) — iDOM's
  commDis expects molecular formula-level DOM tables
- The goal is quantifying assembly processes, dark-matter effects, or DOM-microbe
  networks — use iDOM's commProc, iDME, or H2 instead
- Samples lack a meaningful grouping factor or replication for PERMANOVA — ordination
  alone is descriptive and no significance test is possible
edam_operation: http://edamontology.org/operation_2945
tools:
- name: iDOM
  role: R package providing commDis for Jaccard/Bray-Curtis/UniFrac dissimilarity
    computation on DOM molecular compositional data; installed from GitHub
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment for iDOM and for vegan-based ordination (metaMDS, cmdscale)
    and PERMANOVA (adonis2) on the commDis matrices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:9a41d8eebfa45f286a7e1f54279fb17771851fa4037e2dbc72e596a48d71d6f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - iDOM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dom-dissimilarity-matrix-computation

## Summary

Computes pairwise sample-by-sample dissimilarity matrices for DOM molecular compositional data (presence/absence and relative abundance from FT-ICR MS) using iDOM's commDis with Jaccard, Bray-Curtis, and UniFrac (molecular-trait-weighted) metrics, then ordination and PERMANOVA testing.

## When to use

Use when you have a sample-by-molecular-formula DOM table (presence/absence plus relative abundance) from FT-ICR MS and need chemical diversity/dissimilarity metrics or a test of whether DOM composition differs between sample groupings.

## When NOT to use

- Input is not FT-ICR MS-derived molecular compositional data (e.g., you have 16S amplicon ASV tables or bulk optical DOM indices like EEM/PARAFAC components) — iDOM's commDis expects molecular formula-level DOM tables
- The goal is quantifying assembly processes, dark-matter effects, or DOM-microbe networks — use iDOM's commProc, iDME, or H2 instead
- Samples lack a meaningful grouping factor or replication for PERMANOVA — ordination alone is descriptive and no significance test is possible

## Inputs

- DOM molecular compositional table (samples x molecular formula presence/absence matrix) from FT-ICR MS characterization
- DOM relative-abundance table (samples x molecular formulae)
- Molecular trait table (e.g., H/C, O/C-derived classes) for UniFrac-weighted dissimilarity
- Grouping/metadata factor per sample (e.g., climatic region: subtropical vs temperate) from the experimental warming dataset

## Outputs

- Three sample-by-sample dissimilarity matrices (Jaccard, Bray-Curtis, trait-weighted UniFrac)
- 2-D ordination coordinates (NMDS or PCoA) per dissimilarity matrix, colored by grouping factor (e.g., Figure 3D-style ordination)
- PERMANOVA statistics (F, R2, p-value) per dissimilarity matrix, exported to e.g. dissimilarity_permanova_results.csv

## How to apply

Install the iDOM R package from GitHub with devtools::install_github("jianjunwang/iDOM") and load the example DOM molecular compositional table (samples x molecular formulae, presence/absence and relative abundance). Use commDis to compute three pairwise sample-by-sample dissimilarity matrices: Jaccard on the presence/absence matrix, Bray-Curtis on the relative-abundance matrix, and UniFrac incorporating molecular trait information (H/C, O/C-derived traits). Project each matrix into two dimensions with non-metric multidimensional scaling (vegan::metaMDS) or PCoA (vegan::cmdscale) and plot samples colored by grouping factor (e.g., subtropical vs temperate climatic region). Run vegan::adonis2 PERMANOVA on each dissimilarity matrix against the grouping factor; report F, R2 and p-value, treating p > 0.05 as composition being statistically similar between groups.

## Related tools

- **iDOM** (R package providing commDis for Jaccard/Bray-Curtis/UniFrac dissimilarity computation on DOM molecular compositional data; installed from GitHub) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment for iDOM and for vegan-based ordination (metaMDS, cmdscale) and PERMANOVA (adonis2) on the commDis matrices)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); D_jac <- iDOM::commDis(pa_table, method = "jaccard"); vegan::adonis2(D_jac ~ region, data = meta, permutations = 999)
```

## Evaluation signals

- Each dissimilarity matrix must be square, symmetric, with zero diagonal and sample-count-by-sample-count dimensions matching the input table
- Ordination (NMDS/PCoA) of the dissimilarity matrices should reproduce the article's Figure 3D pattern with samples colored by subtropical vs temperate region
- PERMANOVA (vegan::adonis2 with permutations) on the region factor must yield p > 0.05, confirming no significant compositional difference between subtropical and temperate regions
- All three metrics (Jaccard on presence/absence, Bray-Curtis on relative abundance, UniFrac with molecular traits) should be computed and compared, not just one
- Dissimilarity matrices, ordination coordinates, and PERMANOVA statistics (F, R2, p) written to dissimilarity_permanova_results.csv with complete values

## Limitations

- Dissimilarity assessment is a basic function only; iDOM's advanced analyses (commProc, iDME, H2, iCER) cover different questions and should not be conflated with commDis
- UniFrac here incorporates molecular trait information, so trait-table quality/coverage affects that metric; absence of trait data limits the trait-weighted variant
- iDOM is a development-version GitHub install (zip/tar.gz source install also supported via install.packages with repos = NULL, type = "source"); no CRAN release implies version-drift risk

## Evidence

- [abstract] compositional analyses of chemical diversity and dissimilarity: "the assignment of molecular classes, and the compositional analyses of chemical diversity and dissimilarity"
- [methods] iDOM facilitates evaluation of chemical diversity and dissimilarity: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [readme] Installation of iDOM from GitHub: "devtools::install_github("jianjunwang/iDOM")"
- [readme] iDOM as comprehensive DOM statistical pipeline: "We expect that iDOM will serve as a comprehensive pipeline for DOM statistical analyses and bridge the gap between chemical characterization and ecological interpretation."
