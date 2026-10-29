---
name: betanti-null-model-computation
description: Use when FT-ICR MS-derived DOM molecular compositional data (e.g. MCD,
  TD, or TWCD molecular dendrograms) must be tested for deterministic versus stochastic
  assembly via betaNTI null modeling.
when_to_use_negative:
- Input is raw FT-ICR MS peak lists not yet converted to molecular compositional data
  or trait-based dendrograms — molecular class assignment and trait calculation must
  precede betaNTI null modeling
- The goal is quantifying dark matter effects on DOM interactions (use iDME) or DOM-microbe
  network specialization (use H2) rather than assembly processes
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the commProc function that quantifies the relative influences
    of deterministic and stochastic processes in DOM assemblage assembly via dendrogram-based
    betaNTI null modeling
  repo: https://github.com/jianjunwang/iDOM
- name: commProc
  role: iDOM function executed on molecular dendrograms to compute pairwise betaNTI
    values by comparing observed compositional turnover against null-model randomizations
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install the development version of iDOM from GitHub
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM/commProc betaNTI computation is performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:b8049b7dc149cb862e5cb21a0ff359f0f3827fa259ab065b890a1d77753e6258
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - commProc
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# betanti-null-model-computation

## Summary

Computes dendrogram-based beta nearest taxon index (betaNTI) values for dissolved organic matter (DOM) assemblages using the commProc function of the iDOM R package, quantifying whether deterministic or stochastic processes govern molecular assembly of FT-ICR MS-characterized DOM.

## When to use

Use when FT-ICR MS-derived DOM molecular compositional data (e.g. MCD, TD, or TWCD molecular dendrograms) must be tested for deterministic versus stochastic assembly via betaNTI null modeling.

## When NOT to use

- Input is raw FT-ICR MS peak lists not yet converted to molecular compositional data or trait-based dendrograms — molecular class assignment and trait calculation must precede betaNTI null modeling
- The goal is quantifying dark matter effects on DOM interactions (use iDME) or DOM-microbe network specialization (use H2) rather than assembly processes

## Inputs

- Molecular dendrogram encoding molecular trait-based relatedness of DOM formulae (e.g. MCD, TD, TWCD dendrograms shipped as example datasets with iDOM)
- DOM molecular compositional data (presence/absence or relative abundance of molecular formulae from FT-ICR MS)

## Outputs

- Pairwise betaNTI values for all sample-pair comparisons per dendrogram
- Assembly-process classification per pairwise comparison (deterministic variable selection, homogeneous selection, stochastic/dispersal-associated)
- Summary table of percentage of pairwise comparisons with betaNTI > 2 per dendrogram/fraction (e.g. commProc_betaNTI_results.csv)

## How to apply

Install the development version of iDOM from GitHub with devtools::install_github('jianjunwang/iDOM'). Load the molecular dendrograms (e.g. MCD, TD, TWCD) that encode molecular trait-based relatedness of DOM formulae and run the commProc function, which compares observed compositional turnover against null-model randomizations of the dendrogram to compute pairwise betaNTI values. Classify each pairwise comparison using the betaNTI threshold of 2: values exceeding 2 indicate deterministic variable selection, values below -2 indicate homogeneous selection, and intermediate values indicate stochastic/dispersal-associated processes. Summarize per dendrogram (and per fraction) the percentage of pairwise comparisons with betaNTI > 2 to rank deterministic versus stochastic dominance. In the source study, the majority of betaNTI values for MCD, TD, and TWCD exceeded 2, showing deterministic variable selection dominated, with stochastic processes relatively more important in inactive fractions and homogeneous selection minimal.

## Related tools

- **iDOM** (R package providing the commProc function that quantifies the relative influences of deterministic and stochastic processes in DOM assemblage assembly via dendrogram-based betaNTI null modeling) — https://github.com/jianjunwang/iDOM
- **commProc** (iDOM function executed on molecular dendrograms to compute pairwise betaNTI values by comparing observed compositional turnover against null-model randomizations) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the development version of iDOM from GitHub) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM/commProc betaNTI computation is performed)

## Examples

```
# install.packages("devtools"); devtools::install_github("jianjunwang/iDOM"); library(iDOM); results <- commProc(dendrogram = MCD, comm = dom_composition)
```

## Evaluation signals

- Majority of pairwise betaNTI values exceed the threshold of 2 for MCD, TD, and TWCD dendrograms, indicating deterministic (variable selection) dominance
- Stochastic processes are relatively more important in inactive fractions than in active fractions
- Homogeneous selection (betaNTI < -2) accounts for a minimal fraction of pairwise comparisons
- Results are consistent with the Figure 4A pattern of the source study
- Summary of per-dendrogram/per-fraction percentages is written to commProc_betaNTI_results.csv

## Limitations

- betaNTI requires dendrograms encoding molecular trait-based relatedness; results depend on the dendrogram chosen (MCD, TD, or TWCD)
- Assembly-process classification is threshold-based (betaNTI = 2), so comparisons near the cutoff are ambiguous between deterministic and stochastic categories
- Installation requires the GitHub development version of iDOM (or local zip/tar.gz source install with repos = NULL, type = 'source')

## Evidence

- [methods] commProc quantifies deterministic vs stochastic DOM assembly: "commProc: Quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages."
- [readme] iDOM installed from GitHub via devtools: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] iDOM handles molecular compositional data and traits: [evidence span withheld — anchor status: not_found]
- [abstract] Advanced function quantifies DOM assembly processes: "advanced functions to quantify DOM assembly processes"
