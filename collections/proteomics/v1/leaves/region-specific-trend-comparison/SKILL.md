---
name: region-specific-trend-comparison
description: Use when a molecular compositional dataset (e.g., FT-ICR MS relative
  abundances) spans a continuous treatment gradient (e.g., six temperature levels)
  replicated across two or more regions, and the goal is to test whether a trend (e.
when_to_use_negative:
- The dataset contains only one region, so no between-region contrast of trends is
  possible.
- The goal is quantifying deterministic vs stochastic DOM assembly processes or dark
  matter effects on molecular interactions — use iDOM's commProc or iDME functions
  instead.
- The treatment variable is categorical rather than a numeric gradient, so linear
  regression of diversity against treatment level is inappropriate.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the molecular-trait calculations, chemical diversity functions
    (richness, Shannon, evenness, RaoQ via CWM), and MNTD routines used to compute
    the indices regressed against temperature
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Analysis environment in which iDOM is installed and the per-region linear
    regressions and p-value extraction are performed
- name: devtools
  role: Used to install iDOM from GitHub before running the diversity computations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:503aeab79a416e49e83c43a6080cfa5857a5cc6e621228a474ec7383a0d3bad2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# region-specific-trend-comparison

## Summary

Fits linear regressions of DOM molecular diversity indices (richness, Shannon, evenness, RaoQ, MNTD) against an experimental temperature gradient separately for contrasting regions (temperate vs subtropical), then contrasts the region-specific slopes and p-values to detect divergent warming responses.

## When to use

Use when a molecular compositional dataset (e.g., FT-ICR MS relative abundances) spans a continuous treatment gradient (e.g., six temperature levels) replicated across two or more regions, and the goal is to test whether a trend (e.g., diversity vs temperature) is significant in one region but not another.

## When NOT to use

- The dataset contains only one region, so no between-region contrast of trends is possible.
- The goal is quantifying deterministic vs stochastic DOM assembly processes or dark matter effects on molecular interactions — use iDOM's commProc or iDME functions instead.
- The treatment variable is categorical rather than a numeric gradient, so linear regression of diversity against treatment level is inappropriate.

## Inputs

- FT-ICR MS molecular compositional table with relative abundances of assigned DOM molecules
- Sample metadata assigning each sample to a region (temperate vs subtropical) and a numeric temperature level (six levels in the warming experiment)
- Molecular trait table for assigned molecules (for CWM-based RaoQ and trait-based dendrogram/MNTD)

## Outputs

- Per-sample diversity index values (richness, Shannon, evenness, RaoQ, MNTD) at each temperature level and region
- Per-region regression slopes and p-values for each diversity index vs temperature
- Results table written to diversity_temperature_trends.csv reproducing Figure 3A-C trends

## How to apply

First compute per-sample molecular diversity indices on the FT-ICR MS compositional table at each temperature level: taxonomic-equivalent richness, Shannon, and evenness via iDOM's basic diversity functions; functional diversity as RaoQ via abundance-weighted CWM of molecular traits; and MNTD from a trait-based molecular dendrogram. Then, for each diversity index, fit a linear regression of the index against the numeric temperature level separately within each region (temperate and subtropical) rather than pooling regions, and extract the slope and p-value per region. Apply a p < 0.05 significance threshold to declare a significant trend per region, and contrast regions by whether their slopes are significant (e.g., richness decreases significantly with temperature in the temperate region but not the subtropical, while RaoQ and MNTD decrease significantly in the subtropical but not the temperate). Assemble index values, slopes, and p-values into a per-region results table (diversity_temperature_trends.csv) and validate against the expected pattern of Figure 3A-C.

## Related tools

- **iDOM** (R package providing the molecular-trait calculations, chemical diversity functions (richness, Shannon, evenness, RaoQ via CWM), and MNTD routines used to compute the indices regressed against temperature) — https://github.com/jianjunwang/iDOM
- **R** (Analysis environment in which iDOM is installed and the per-region linear regressions and p-value extraction are performed)
- **devtools** (Used to install iDOM from GitHub before running the diversity computations)

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Molecular richness regressed against temperature must yield a significant negative slope in the temperate region (p < 0.05) but a non-significant slope in the subtropical region (p > 0.05), matching Figure 3A.
- RaoQ and MNTD must decrease significantly with temperature in the subtropical region (p < 0.05) but not in the temperate region, matching Figure 3A-C.
- The output CSV (diversity_temperature_trends.csv) must contain, for each diversity index and each region, the index values, the regression slope, and the p-value.
- All regressions must be fitted separately per region, not on pooled data — pooled fits would mask the region-specific significance contrast.

## Limitations

- Linear regression assumes a monotonic, approximately linear response of each diversity index to temperature; non-linear thermal responses would not be captured.
- Significance is declared only at the p < 0.05 threshold; marginal p-values near this cutoff should be interpreted cautiously.
- The demonstrated pattern (richness declining in the temperate region, RaoQ/MNTD declining in the subtropical) derives from the bundled experimental warming dataset and may not generalize to other regions or DOM assemblages.

## Evidence

- [results] DOM molecular richness decreased significantly with rising temperature in the temperate region (p < 0.05): [evidence span withheld — anchor status: not_found]
- [abstract] analyze the ecological networks between DOM and microbes, and explore their response to warming: "analyze the ecological networks between DOM and microbes, and explore their response to warming"
- [methods] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [readme] Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
