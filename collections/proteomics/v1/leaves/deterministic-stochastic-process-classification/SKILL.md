---
name: deterministic-stochastic-process-classification
description: Use when FT-ICR MS-derived DOM compositional data and molecular dendrograms
  (e.g., MCD, TD, TWCD) are available and the goal is to classify pairwise assembly
  turnover as deterministic variable selection, homogeneous selection, or stochastic
  using betaNTI thresholds.
when_to_use_negative:
- Input lacks a molecular dendrogram or trait-based relatedness structure of DOM formulae,
  so dendrogram-based betaNTI null modeling cannot be computed
- The goal is analysis of taxonomic (microbial) community assembly rather than DOM
  molecular assemblage assembly
- Only basic DOM statistics are needed (molecular traits, molecular class assignment,
  chemical diversity/dissimilarity), which are handled by iDOM's basic functions instead
  of commProc
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3299
tools:
- name: iDOM
  role: R package providing the commProc function that quantifies the relative influences
    of deterministic and stochastic processes in the assembly of DOM assemblages
  repo: https://github.com/jianjunwang/iDOM
- name: commProc
  role: iDOM function that computes pairwise betaNTI via dendrogram-based null modeling
    and classifies DOM assembly processes
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
  source_content_hash: sha256:e69c27e4d61b27f0bf5e97a25df6689e2515a7b161f79df94c56d425b5a549c1
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
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# deterministic-stochastic-process-classification

## Summary

Quantifies whether deterministic or stochastic processes dominate the assembly of dissolved organic matter (DOM) assemblages using dendrogram-based betaNTI null modeling via the commProc function in the iDOM R package.

## When to use

Use when FT-ICR MS-derived DOM compositional data and molecular dendrograms (e.g., MCD, TD, TWCD) are available and the goal is to classify pairwise assembly turnover as deterministic variable selection, homogeneous selection, or stochastic using betaNTI thresholds.

## When NOT to use

- Input lacks a molecular dendrogram or trait-based relatedness structure of DOM formulae, so dendrogram-based betaNTI null modeling cannot be computed
- The goal is analysis of taxonomic (microbial) community assembly rather than DOM molecular assemblage assembly
- Only basic DOM statistics are needed (molecular traits, molecular class assignment, chemical diversity/dissimilarity), which are handled by iDOM's basic functions instead of commProc

## Inputs

- Molecular dendrogram of DOM formulae (e.g., MCD, TD, or TWCD) encoding molecular trait-based relatedness from FT-ICR MS data
- DOM molecular compositional data (presence/absence or relative abundance of formulae across samples/fractions)

## Outputs

- Pairwise betaNTI values for all sample comparisons per dendrogram
- Per-comparison assembly process classification (variable selection, homogeneous selection, stochastic/dispersal-associated)
- Summary table (e.g., commProc_betaNTI_results.csv) with percentage of betaNTI > 2 per dendrogram and per fraction and dominance ranking of deterministic vs. stochastic processes

## How to apply

Install iDOM from GitHub with devtools::install_github('jianjunwang/iDOM'), then load the MCD, TD, and TWCD molecular dendrograms encoding trait-based relatedness of DOM formulae. Run the commProc function on each dendrogram to compute pairwise betaNTI values, comparing observed compositional turnover against null-model randomizations of the dendrogram. Classify each pairwise comparison using the betaNTI threshold of 2: values exceeding 2 indicate deterministic variable selection, values below -2 indicate homogeneous selection, and values between -2 and 2 indicate stochastic processes. Summarize, per dendrogram and per fraction, the percentage of pairwise comparisons with betaNTI > 2 and rank deterministic versus stochastic dominance, writing results to a summary table such as commProc_betaNTI_results.csv.

## Related tools

- **iDOM** (R package providing the commProc function that quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages) — https://github.com/jianjunwang/iDOM
- **commProc** (iDOM function that computes pairwise betaNTI via dendrogram-based null modeling and classifies DOM assembly processes) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the development version of iDOM from GitHub) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM and commProc are executed)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); library(iDOM); results <- commProc(dendrogram = MCD, output = 'commProc_betaNTI_results.csv')
```

## Evaluation signals

- Majority of betaNTI values exceed 2 for MCD, TD, and TWCD dendrograms, indicating dominance of deterministic variable selection (consistent with Figure 4A)
- Stochastic processes are relatively more important in inactive fractions
- Homogeneous selection is minimal (few betaNTI values below -2)
- All pairwise comparisons are classified into exactly one category using the |betaNTI| = 2 threshold
- A summary table (commProc_betaNTI_results.csv) is written with per-dendrogram and per-fraction percentages

## Limitations

- Classification depends on the quality of the molecular dendrogram (MCD, TD, or TWCD); dendrogram choice changes the trait-based relatedness structure and thus betaNTI values
- Inference is relative to the null model randomization scheme; results reflect turnover patterns captured by the dendrogram, not all possible assembly mechanisms
- The method addresses DOM molecular assemblages characterized by high-resolution mass spectrometry (FT-ICR MS); it is not a general taxonomic beta-diversity tool

## Evidence

- [methods] commProc quantifies deterministic vs stochastic DOM assembly via betaNTI with dendrogram null model: "commProc: Quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages."
- [readme] Install iDOM from GitHub with devtools: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] iDOM is an R package for basic and advanced statistical analyses and visualization of DOM data: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
- [abstract] Case studies quantify DOM assembly processes with advanced functions: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions"
- [results] Majority of betaNTI values exceed 2 indicating deterministic variable selection dominates: "The majority of βNTI values for MCD, TD, and TWCD exceeded 2, indicating that deterministic processes, particularly variable selection, predominantly governed molecular assembly (Figure 4A)."
