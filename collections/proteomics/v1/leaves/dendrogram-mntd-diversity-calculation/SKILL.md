---
name: dendrogram-mntd-diversity-calculation
description: Use when molecular compositional data from FT-ICR MS must be converted
  into a dendrogram of molecular relatedness so that MNTD-style functional diversity
  can be computed per assemblage across an experimental gradient.
when_to_use_negative:
- Input is raw FT-ICR MS peak lists that have not yet been converted to a molecular
  compositional table (relative abundances per assigned formula) — trait and dendrogram
  computation presumes processed compositional data
- The analysis goal is phylogenetic diversity of taxa with an actual phylogeny — this
  skill substitutes a trait-based molecular dendrogram, not a true phylogeny
- The goal is quantifying deterministic vs. stochastic DOM assembly processes or dark-matter
  effects on networks — use iDOM's commProc or iDME functions instead
tools:
- name: iDOM
  role: R package providing the molecular-trait calculation, dissimilarity evaluation,
    and diversity routines used to build the trait-based dendrogram and compute MNTD
    and RaoQ
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM's basic and advanced statistical analyses
    of DOM data are executed
- name: devtools
  role: Used to install the iDOM package from its GitHub source before running the
    diversity workflow
  repo: https://github.com/jianjunwang/iDOM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:ef482058084d41aad8e34538ee6efc433c69fd03fdd35f7922343d5a84dc6b56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# dendrogram-mntd-diversity-calculation

## Summary

Builds a molecular dendrogram from trait-based dissimilarities of FT-ICR MS-derived DOM molecules and computes mean nearest taxon distance (MNTD) per assemblage, treating molecules as a phylogeny-equivalent for functional diversity analysis.

## When to use

Use when molecular compositional data from FT-ICR MS must be converted into a dendrogram of molecular relatedness so that MNTD-style functional diversity can be computed per assemblage across an experimental gradient.

## When NOT to use

- Input is raw FT-ICR MS peak lists that have not yet been converted to a molecular compositional table (relative abundances per assigned formula) — trait and dendrogram computation presumes processed compositional data
- The analysis goal is phylogenetic diversity of taxa with an actual phylogeny — this skill substitutes a trait-based molecular dendrogram, not a true phylogeny
- The goal is quantifying deterministic vs. stochastic DOM assembly processes or dark-matter effects on networks — use iDOM's commProc or iDME functions instead

## Inputs

- FT-ICR MS molecular compositional table (molecular formulae with relative abundances per sample, plus metadata such as temperature level and region)
- Molecular trait table for assigned molecules (e.g. element ratios / trait-based descriptors computed by iDOM)

## Outputs

- Molecular dendrogram built from trait-based dissimilarities among molecules
- Per-assemblage MNTD values across the temperature gradient
- RaoQ functional diversity values via CWM of molecular traits
- Per-region regression table of diversity index values, slopes and p-values (e.g. diversity_temperature_trends.csv)

## How to apply

Start from the FT-ICR MS molecular compositional table (molecule assignments with relative abundances, e.g. the iDOM experimental warming dataset spanning six temperature levels in temperate and subtropical regions). Compute molecular traits and trait-based dissimilarities among molecules using iDOM's molecular-trait routines, then cluster the dissimilarity matrix into a molecular dendrogram that serves as a phylogeny-equivalent. For each sample assemblage, compute MNTD on this dendrogram, weighting by the relative abundances of assigned molecules. Alongside MNTD, compute RaoQ functional diversity via community-weighted means of molecular traits, and basic indices (richness, Shannon, evenness) on the compositional data. Finally, regress each diversity index against temperature level separately per region, extract slopes and p-values, and apply a p < 0.05 significance threshold; assemble results (index values, slopes, p-values per region) into a table such as diversity_temperature_trends.csv.

## Related tools

- **iDOM** (R package providing the molecular-trait calculation, dissimilarity evaluation, and diversity routines used to build the trait-based dendrogram and compute MNTD and RaoQ) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM's basic and advanced statistical analyses of DOM data are executed)
- **devtools** (Used to install the iDOM package from its GitHub source before running the diversity workflow) — https://github.com/jianjunwang/iDOM

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Richness must decrease significantly with rising temperature in the temperate region (p < 0.05) but show no significant change in the subtropical region (p > 0.05), matching Figure 3A
- RaoQ and MNTD must decrease significantly in the subtropical region (p < 0.05) but not in the temperate region, matching Figure 3A-C
- The molecular dendrogram must be a valid hierarchical clustering of the trait-based dissimilarity matrix (molecules as tips, relative abundances used as weights)
- The results table contains per-region and per-index rows with columns for index value, regression slope and p-value, and reproduces the region-specific trends of Figure 3A-C

## Limitations

- Functional diversity depends on molecules having assigned molecular classes/traits; uncharacterized molecules (dark matter) are excluded from trait weighting by this workflow
- The molecular dendrogram is a trait-based proxy, so MNTD here reflects functional (chemical) relatedness rather than evolutionary relatedness; conclusions are tied to the chosen trait set and dissimilarity measure
- The workflow's validation criteria are specific to the bundled experimental warming dataset (six temperature levels, temperate vs. subtropical regions); other datasets will not meet the temperate/subtropical significance expectations

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and dissimilarity.: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [abstract] The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter).: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [readme] We expect that iDOM will serve as a comprehensive pipeline for DOM statistical analyses and bridge the gap between chemical characterization and ecological interpretation.: "We expect that iDOM will serve as a comprehensive pipeline for DOM statistical analyses and bridge the gap between chemical characterization and ecological interpretation."
