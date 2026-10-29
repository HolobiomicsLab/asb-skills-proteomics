---
name: ordination-plot-visualization
description: Use when sample-by-sample dissimilarity matrices computed from FT-ICR
  MS DOM molecular compositional data (presence/absence or relative abundance) must
  be projected into 2D ordination plots colored by a grouping factor such as subtropical
  vs temperate region.
when_to_use_negative:
- Input is raw FT-ICR MS peak lists rather than a processed samples x formula compositional
  table — molecular traits and class assignment must be computed first
- The analysis goal is DOM assembly process quantification (commProc) or dark-matter
  network effects (iDME) rather than compositional dissimilarity visualization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: iDOM
  role: R package whose commDis function computes the Jaccard, Bray–Curtis, and UniFrac
    dissimilarity matrices that feed the ordination
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Environment for installing iDOM from GitHub and running vegan metaMDS/cmdscale
    ordination and adonis2 PERMANOVA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:6d9298db731ae354b24c7830033987f2aba9b9d35d5af225e9ed75a5a411d62c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - iDOM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# ordination-plot-visualization

## Summary

Projects iDOM commDis-derived DOM dissimilarity matrices (Jaccard, Bray–Curtis, UniFrac) into 2D ordination space using NMDS (vegan::metaMDS) or PCoA (vegan::cmdscale) to visualize molecular compositional patterns, then validates group separation with vegan::adonis2 PERMANOVA.

## When to use

Use when sample-by-sample dissimilarity matrices computed from FT-ICR MS DOM molecular compositional data (presence/absence or relative abundance) must be projected into 2D ordination plots colored by a grouping factor such as subtropical vs temperate region.

## When NOT to use

- Input is raw FT-ICR MS peak lists rather than a processed samples x formula compositional table — molecular traits and class assignment must be computed first
- The analysis goal is DOM assembly process quantification (commProc) or dark-matter network effects (iDME) rather than compositional dissimilarity visualization

## Inputs

- DOM molecular compositional table (samples x molecular formula presence/absence) from FT-ICR MS data
- DOM molecular compositional table (samples x molecular formula relative abundance)
- iDOM commDis dissimilarity matrices (Jaccard, Bray–Curtis, UniFrac)
- Grouping factor vector (e.g., subtropical vs temperate climatic region per sample)

## Outputs

- 2D ordination coordinates per sample (NMDS or PCoA) from each dissimilarity matrix
- Ordination plots of samples colored by climatic region (Figure 3D-style)
- PERMANOVA statistics (F, R2, p-value) per dissimilarity matrix, written to dissimilarity_permanova_results.csv

## How to apply

First compute the pairwise sample dissimilarity matrices with iDOM's commDis: Jaccard on the formula presence/absence table, Bray–Curtis on the relative-abundance table, and UniFrac incorporating molecular trait information. Then project each matrix into two dimensions with non-metric multidimensional scaling (vegan::metaMDS) or principal coordinates analysis (vegan::cmdscale) and plot samples colored by the climatic-region grouping factor to reproduce the paper's Figure 3D ordination. Finally run PERMANOVA (vegan::adonis2 with permutations) on the same dissimilarity matrices against the grouping factor to confirm whether the ordination separation is statistically significant; in the warming case study the subtropical vs temperate comparison must return p > 0.05.

## Related tools

- **iDOM** (R package whose commDis function computes the Jaccard, Bray–Curtis, and UniFrac dissimilarity matrices that feed the ordination) — https://github.com/jianjunwang/iDOM
- **R** (Environment for installing iDOM from GitHub and running vegan metaMDS/cmdscale ordination and adonis2 PERMANOVA)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); ord <- vegan::metaMDS(D_jaccard); ordplot(ord, color=region); vegan::adonis2(D_jaccard ~ region, permutations=999)
```

## Evaluation signals

- Ordination compresses each commDis matrix to 2 dimensions without excessive stress (check NMDS stress from metaMDS)
- PERMANOVA p-value for the subtropical/temperate region factor exceeds 0.05, confirming molecular composition is statistically similar between the two climate zones
- All three dissimilarity types (Jaccard on presence/absence, Bray–Curtis on abundance, UniFrac with molecular traits) are ordinated and plotted separately
- Dissimilarity matrices, ordination coordinates, and PERMANOVA F/R2/p statistics are exported to dissimilarity_permanova_results.csv
- Ordination plot colors samples by the climatic-region grouping factor as in Figure 3D

## Limitations

- Interpretation of ordination separation must be backed by PERMANOVA — visual clustering alone is not a significance test
- The p > 0.05 result applies to the experimental warming example dataset's subtropical vs temperate contrast; other datasets may differ
- UniFrac requires molecular trait information in addition to presence/absence data; it cannot be computed from compositional tables alone

## Evidence

- [abstract] the compositional analyses of chemical diversity and dissimilarity: "the assignment of molecular classes, and the compositional analyses of chemical diversity and dissimilarity"
- [methods] the evaluation of chemical diversity and dissimilarity: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [readme] the evaluation of chemical diversity and dissimilarity (README): "basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and dissimilarity"
