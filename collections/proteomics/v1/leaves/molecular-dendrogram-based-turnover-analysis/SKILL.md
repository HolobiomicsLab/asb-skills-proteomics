---
name: molecular-dendrogram-based-turnover-analysis
description: Use when FT-ICR MS-derived DOM molecular composition data are available
  and the goal is to classify pairwise assemblage turnover as deterministic (variable/homogeneous
  selection) or stochastic using a betaNTI null-model threshold of 2.
when_to_use_negative:
- When the input is raw FT-ICR MS spectra rather than processed molecular compositional
  data or molecular dendrograms — molecular traits and class assignment must be computed
  first with iDOM's basic functions
- When the ecological question concerns dark matter effects on molecular interactions
  (use iDME) or DOM-microbe network specialization (use H2) rather than assemblage
  assembly processes
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the commProc function for dendrogram-based betaNTI null
    modeling of DOM assemblage assembly
  repo: https://github.com/jianjunwang/iDOM
- name: commProc
  role: iDOM function that quantifies the relative influences of deterministic and
    stochastic processes in DOM assemblage assembly via betaNTI
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install the development version of iDOM from GitHub
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM and commProc are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:7159e110c36f3c498ad66938934048f30fe25849ff3eaeb4b03955e48f8d8f9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - commProc
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# molecular-dendrogram-based-turnover-analysis

## Summary

Quantifies the relative influences of deterministic versus stochastic processes in the assembly of dissolved organic matter (DOM) assemblages by computing dendrogram-based betaNTI with the iDOM commProc function on molecular dendrograms (e.g., MCD, TD, TWCD) derived from FT-ICR MS molecular data.

## When to use

Use when FT-ICR MS-derived DOM molecular composition data are available and the goal is to classify pairwise assemblage turnover as deterministic (variable/homogeneous selection) or stochastic using a betaNTI null-model threshold of 2.

## When NOT to use

- When the input is raw FT-ICR MS spectra rather than processed molecular compositional data or molecular dendrograms — molecular traits and class assignment must be computed first with iDOM's basic functions
- When the ecological question concerns dark matter effects on molecular interactions (use iDME) or DOM-microbe network specialization (use H2) rather than assemblage assembly processes

## Inputs

- Molecular dendrograms of DOM formulae encoding molecular trait-based relatedness (e.g., MCD, TD, TWCD) from FT-ICR MS molecular compositional data
- DOM molecular compositional data (presence/absence or relative abundance of molecular formulae) across samples/fractions

## Outputs

- Pairwise betaNTI values for each comparison of DOM assemblages
- Per-dendrogram and per-fraction summary of the percentage of pairwise comparisons with betaNTI > 2 and dominance ranking of deterministic vs stochastic processes
- Summary table written to commProc_betaNTI_results.csv

## How to apply

Install the development version of iDOM from GitHub with devtools::install_github('jianjunwang/iDOM') in R. Load molecular dendrograms of the example datasets shipped with iDOM (MCD, TD, TWCD), which encode molecular trait-based relatedness of DOM formulae. Run commProc on each dendrogram: it compares observed compositional turnover against null-model randomizations of the dendrogram to compute pairwise betaNTI values. Classify each pairwise comparison using the betaNTI threshold of 2 — values > 2 indicate deterministic variable selection, values < -2 homogeneous selection, and values between -2 and 2 stochastic/dispersal-associated processes. Summarize, per dendrogram and per fraction, the percentage of pairwise comparisons with betaNTI exceeding 2 and the dominance ranking of deterministic versus stochastic processes, writing the summary to commProc_betaNTI_results.csv.

## Related tools

- **iDOM** (R package providing the commProc function for dendrogram-based betaNTI null modeling of DOM assemblage assembly) — https://github.com/jianjunwang/iDOM
- **commProc** (iDOM function that quantifies the relative influences of deterministic and stochastic processes in DOM assemblage assembly via betaNTI) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the development version of iDOM from GitHub) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM and commProc are executed)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); library(iDOM); betaNTI_results <- commProc(dendrogram = MCD, abund = DOM_composition); write.csv(betaNTI_results, 'commProc_betaNTI_results.csv')
```

## Evaluation signals

- The majority of betaNTI values for MCD, TD, and TWCD exceed 2, consistent with the Figure 4A pattern where deterministic processes, particularly variable selection, dominate
- Stochastic processes are relatively more important in inactive fractions and homogeneous selection is minimal
- The summary table commProc_betaNTI_results.csv is produced with per-dendrogram, per-fraction percentages of betaNTI > 2
- Each pairwise comparison is classified into assembly categories using the betaNTI threshold of 2 (deterministic variable selection, homogeneous selection, stochastic/dispersal-associated)
- iDOM installs successfully from GitHub via devtools::install_github('jianjunwang/iDOM') and the shipped example dendrograms load correctly

## Limitations

- Requires precomputed molecular dendrograms (MCD, TD, TWCD) derived from FT-ICR MS data; results depend on how molecular trait-based relatedness of DOM formulae is encoded in the dendrogram
- Interpretation is threshold-based (betaNTI of 2), so borderline values near the cutoffs yield ambiguous process classifications
- Published as a development version installed from GitHub; no CRAN release or explicit license is stated in the README

## Evidence

- [methods] commProc: Quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages.: "commProc: Quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages."
- [readme] devtools install: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] advanced functions quantify assembly: "advanced functions to quantify DOM assembly processes"
- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [results] The majority of βNTI values for MCD, TD, and TWCD exceeded 2, indicating that deterministic processes, particularly variable selection, predominantly governed molecular assembly (Figure 4A).: "The majority of βNTI values for MCD, TD, and TWCD exceeded 2, indicating that deterministic processes, particularly variable selection, predominantly governed molecular assembly"
