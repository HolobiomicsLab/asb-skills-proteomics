---
name: temperature-gradient-trend-regression
description: Use when a FT-ICR MS molecular compositional table sampled across multiple
  temperature levels (e.g., six levels in temperate and subtropical regions) must
  be tested for significant diversity-temperature trends.
when_to_use_negative:
- The data are already a taxon/OTU feature table rather than an FT-ICR MS molecular
  compositional table — this skill targets DOM molecules, not taxonomy
- Temperature is not a designed experimental gradient (no replicate levels), so a
  linear regression against temperature level is not meaningful
- The goal is quantifying assembly processes or dark-matter effects rather than testing
  thermal response trends — use commProc or iDME instead
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: iDOM
  role: R package supplying the molecular-trait, molecular-class-assignment, chemical
    diversity and advanced functions used to compute the indices regressed against
    temperature
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Language in which iDOM runs and in which the per-region linear regressions
    and p-value extraction are performed
- name: devtools
  role: Used to install iDOM from its GitHub development repository
- name: FT-ICR MS
  role: Instrument producing the high-resolution molecular compositional data (relative
    abundances of DOM molecules) that form the regression input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:d18b3a7c39d99e3d4b1a4c2098e56fab8bcbf67ab75f49a0896195cd64539ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# temperature-gradient-trend-regression

## Summary

Fits linear regressions of DOM molecular diversity indices (richness, Shannon, evenness, RaoQ functional diversity, MNTD) against experimental temperature levels, separately per region (temperate vs subtropical), to test whether dissolved organic matter molecular richness changes significantly with warming.

## When to use

Use when a FT-ICR MS molecular compositional table sampled across multiple temperature levels (e.g., six levels in temperate and subtropical regions) must be tested for significant diversity-temperature trends.

## When NOT to use

- The data are already a taxon/OTU feature table rather than an FT-ICR MS molecular compositional table — this skill targets DOM molecules, not taxonomy
- Temperature is not a designed experimental gradient (no replicate levels), so a linear regression against temperature level is not meaningful
- The goal is quantifying assembly processes or dark-matter effects rather than testing thermal response trends — use commProc or iDME instead

## Inputs

- FT-ICR MS molecular compositional table with relative abundances of assigned DOM molecules per sample (e.g., iDOM's bundled experimental warming dataset)
- Sample metadata mapping each sample to one of six temperature levels and to a region (temperate or subtropical)
- Molecular trait table for assigned molecules (used for CWM/RaoQ and trait-based dissimilarity)

## Outputs

- Per-sample diversity index values (richness, Shannon, evenness, RaoQ, MNTD) at each temperature level
- Regression slopes and p-values per diversity index per region
- Results table (e.g., diversity_temperature_trends.csv) reproducing Figure 3A-C trends

## How to apply

Compute per-sample diversity indices on the molecular compositional data at each temperature level using iDOM's basic diversity functions: taxonomic-equivalent indices (richness, Shannon, evenness), RaoQ functional diversity via CWM weighted by relative abundances of assigned molecules, and MNTD from a molecular dendrogram built on trait-based dissimilarities. Then fit linear regressions of each index against temperature level separately for the temperate and subtropical regions, extracting slopes and p-values with a p < 0.05 significance threshold. The region-stratified design is essential because the expected trends are region-specific: richness decreases significantly with temperature only in the temperate region, while RaoQ and MNTD decrease significantly only in the subtropical region. Assemble index values, slopes and p-values per region into a results table (e.g., diversity_temperature_trends.csv) that reproduces the region-specific trends of Figure 3A-C.

## Related tools

- **iDOM** (R package supplying the molecular-trait, molecular-class-assignment, chemical diversity and advanced functions used to compute the indices regressed against temperature) — https://github.com/jianjunwang/iDOM
- **R** (Language in which iDOM runs and in which the per-region linear regressions and p-value extraction are performed)
- **devtools** (Used to install iDOM from its GitHub development repository)
- **FT-ICR MS** (Instrument producing the high-resolution molecular compositional data (relative abundances of DOM molecules) that form the regression input)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); fit <- lm(richness ~ temperature, data = dom, subset = region == "temperate"); summary(fit)$coefficients
```

## Evaluation signals

- Molecular richness must decrease significantly with rising temperature in the temperate region (p < 0.05) but show no significant change in the subtropical region (p > 0.05), matching Figure 3A
- RaoQ and MNTD must decrease significantly in the subtropical region (p < 0.05) but not in the temperate region, matching Figure 3A-C
- Every sample is assigned to exactly one of six temperature levels and one region before regression; no sample is dropped from the per-region fits
- The output table contains index value, slope and p-value for each diversity index × region combination
- Regressions must be fit per region (separate temperate and subtropical models), not on pooled data

## Limitations

- Findings are region-specific: extrapolating the temperate richness decline or subtropical RaoQ/MNTD decline to other regions or field settings is unsupported
- Significance relies on a p < 0.05 threshold on linear regression of index vs. temperature level; index choice (taxonomic-equivalent vs. functional vs. phylogenetic-equivalent MNTD) determines which trends are detected
- Indices depend on molecules assigned by FT-ICR MS; uncharacterized molecules (dark matter) can affect molecular interactions and are handled by separate iDOM functions (iDME), not this regression

## Evidence

- [abstract] advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM"
- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [methods] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [readme] iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
