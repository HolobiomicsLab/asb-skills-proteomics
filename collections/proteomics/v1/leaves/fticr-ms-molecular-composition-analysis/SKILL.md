---
name: fticr-ms-molecular-composition-analysis
description: Use when FT-ICR MS yields a samples-by-molecular-formula compositional
  table (presence/absence plus relative abundance) and the goal is molecular traits,
  class assignment, diversity indices, dissimilarity matrices, or region/temperature
  comparisons of DOM composition.
when_to_use_negative:
- Input is raw FT-ICR MS peak/feature output that has not yet been converted to a
  samples-by-formula compositional table — molecular traits and class assignment in
  iDOM operate on compositional data, not spectra
- The goal is quantifying DOM assembly processes (commProc), dark-matter co-occurrence
  effects (iDME), or DOM-microbe bipartite network specialization (H2) — these are
  separate advanced iDOM functions outside this composition/diversity skill
- Data are taxonomic (ASV/OTU) rather than molecular formula composition; iDOM's chemical
  diversity and dissimilarity functions target FT-ICR MS DOM data
tools:
- name: iDOM
  role: 'Core R package providing molecular trait calculation, molecular class assignment,
    chemical diversity (richness, Shannon, evenness, RaoQ, MNTD) and dissimilarity
    (commDis: Jaccard, Bray-Curtis, UniFrac) functions applied to FT-ICR MS DOM data'
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM functions are executed and regression/PERMANOVA
    analyses are scripted
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install iDOM from its GitHub repository via devtools::install_github("jianjunwang/iDOM")
  repo: https://github.com/jianjunwang/iDOM
- name: vegan
  role: Provides metaMDS/cmdscale for ordination of commDis dissimilarity matrices
    and adonis2 for PERMANOVA testing against the subtropical/temperate grouping factor
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:f5e5abd6c2e25b15925fb18313130c36d9bc50fd6f7676968fb93f4ad3e3e75d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fticr-ms-molecular-composition-analysis

## Summary

Analyzes dissolved organic matter (DOM) molecular composition from FT-ICR MS using the iDOM R package: computing molecular traits, assigning molecular classes, and quantifying chemical diversity (richness, Shannon, evenness, RaoQ, MNTD) and sample-by-sample dissimilarity (Jaccard, Bray-Curtis, UniFrac) for downstream ordination and PERMANOVA testing.

## When to use

Use when FT-ICR MS yields a samples-by-molecular-formula compositional table (presence/absence plus relative abundance) and the goal is molecular traits, class assignment, diversity indices, dissimilarity matrices, or region/temperature comparisons of DOM composition.

## When NOT to use

- Input is raw FT-ICR MS peak/feature output that has not yet been converted to a samples-by-formula compositional table — molecular traits and class assignment in iDOM operate on compositional data, not spectra
- The goal is quantifying DOM assembly processes (commProc), dark-matter co-occurrence effects (iDME), or DOM-microbe bipartite network specialization (H2) — these are separate advanced iDOM functions outside this composition/diversity skill
- Data are taxonomic (ASV/OTU) rather than molecular formula composition; iDOM's chemical diversity and dissimilarity functions target FT-ICR MS DOM data

## Inputs

- FT-ICR MS molecular compositional table (samples x molecular formula presence/absence and relative abundance), e.g. the iDOM bundled experimental warming dataset with six temperature levels and subtropical vs temperate regions
- Grouping factors (climatic region, temperature level)
- Molecular traits assigned to formulas (for RaoQ via CWM and trait-based UniFrac)

## Outputs

- Per-sample molecular diversity indices (richness, Shannon, evenness) across temperature levels
- Functional diversity (RaoQ) and phylogenetic-equivalent diversity (MNTD from molecular dendrogram)
- Three sample-by-sample dissimilarity matrices (Jaccard, Bray-Curtis, UniFrac) from commDis
- NMDS or PCoA two-dimensional ordination coordinates colored by climatic region (Figure 3D)
- PERMANOVA statistics (F, R2, p-value) from vegan::adonis2
- Results tables such as diversity_temperature_trends.csv (index values, slopes, p-values per region) and dissimilarity_permanova_results.csv

## How to apply

Install iDOM from GitHub with devtools::install_github("jianjunwang/iDOM") and load the bundled experimental warming dataset (FT-ICR MS molecular compositional table with relative abundances, six temperature levels, temperate and subtropical regions). For diversity trends, compute per-sample molecular diversity indices (richness, Shannon, evenness) at each temperature level, functional diversity as RaoQ via CWM weighted by relative abundances of assigned molecules, and MNTD from a molecular dendrogram built on trait-based dissimilarities; fit linear regressions of each index against temperature per region with a p < 0.05 threshold. For compositional dissimilarity, compute three pairwise sample-by-sample matrices with iDOM's commDis: Jaccard on presence/absence, Bray-Curtis on relative abundance, and UniFrac incorporating molecular traits. Project each matrix into two dimensions with NMDS (vegan::metaMDS) or PCoA (vegan::cmdscale) colored by climatic region (Figure 3D), then run vegan::adonis2 PERMANOVA on the matrices against the subtropical/temperate factor, extracting F, R2, and p-value. Decision point: a non-significant region term (p > 0.05) indicates similar DOM composition across climate zones, while significant diversity-temperature slopes (p < 0.05) identify region-specific thermal responses.

## Related tools

- **iDOM** (Core R package providing molecular trait calculation, molecular class assignment, chemical diversity (richness, Shannon, evenness, RaoQ, MNTD) and dissimilarity (commDis: Jaccard, Bray-Curtis, UniFrac) functions applied to FT-ICR MS DOM data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM functions are executed and regression/PERMANOVA analyses are scripted) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install iDOM from its GitHub repository via devtools::install_github("jianjunwang/iDOM")) — https://github.com/jianjunwang/iDOM
- **vegan** (Provides metaMDS/cmdscale for ordination of commDis dissimilarity matrices and adonis2 for PERMANOVA testing against the subtropical/temperate grouping factor)

## Examples

```
install.packages('devtools'); devtools::install_github('jianjunwang/iDOM')
```

## Evaluation signals

- PERMANOVA on commDis-derived dissimilarity matrices returns a non-significant region effect (p > 0.05), matching the reported result that molecular DOM composition is similar between subtropical and temperate regions
- Molecular richness decreases significantly with temperature in the temperate region (p < 0.05) but not in the subtropical region (p > 0.05), reproducing Figure 3A
- RaoQ and MNTD decrease significantly in the subtropical region (p < 0.05) but not in the temperate region, matching Figure 3A-C
- Three distinct dissimilarity matrices are produced (Jaccard on presence/absence, Bray-Curtis on relative abundance, UniFrac with molecular traits) and their NMDS/PCoA ordinations reproduce the Figure 3D grouping of samples by climatic region
- Output tables (e.g. diversity_temperature_trends.csv, dissimilarity_permanova_results.csv) contain per-region index values, slopes, and PERMANOVA F, R2, and p-values

## Limitations

- iDOM must be installed from a GitHub development version (or local zip/tar.gz source with repos = NULL, type = "source"), so installation may be fragile and depends on devtools availability
- UniFrac-style dissimilarity requires molecular trait information from which a dendrogram is built; absence of assigned molecular traits limits commFD/commDD and trait-weighted analyses
- Results interpretation rests on a p < 0.05 significance threshold for region-specific regressions and PERMANOVA terms; borderline p-values do not reproduce the published findings
- The demonstrated case studies use the bundled experimental warming dataset (six temperature levels, two climatic regions); other DOM datasets may require adapting the grouping factors

## Evidence

- [abstract] we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
- [methods] iDOM multifunctional tool: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [abstract] package handles DOM data types: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [readme] README installation: "devtools::install_github("jianjunwang/iDOM")"
- [readme] README chemical diversity scope: "the evaluation of chemical diversity and dissimilarity. It
also includes functions for advanced analyses to quantify the assembly
processes of DOM assemblages"
