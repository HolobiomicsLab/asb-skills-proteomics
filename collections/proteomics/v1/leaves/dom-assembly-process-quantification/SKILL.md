---
name: dom-assembly-process-quantification
description: Use when FT-ICR MS molecular compositional data of dissolved organic
  matter are available and the goal is to classify pairwise turnover between assemblages
  into deterministic (variable/homogeneous selection) versus stochastic assembly processes
  via betaNTI null modeling.
when_to_use_negative:
- The input is not molecular compositional data of DOM and no molecular dendrogram
  (MCD, TD, TWCD or equivalent) of DOM formulae can be constructed, since commProc
  relies on dendrogram-based relatedness of formulae.
- The goal is quantifying dark matter effects on molecular interactions, DOM-microbe
  network specialization, or thermal response of DOM — use iDME, H2, or iCER instead
  of commProc.
- Only a single assemblage (no pairwise comparisons between samples or fractions)
  is available, so betaNTI turnover cannot be computed.
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3452
tools:
- name: iDOM
  role: R package providing the commProc function that quantifies deterministic versus
    stochastic DOM assembly processes via betaNTI null modeling; ships example MCD,
    TD, and TWCD dendrograms
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and commProc is executed on
    the dendrograms and compositional data
- name: devtools
  role: Used to install the development version of iDOM from GitHub (devtools::install_github('jianjunwang/iDOM'))
- name: FT-ICR MS
  role: Instrument generating the high-resolution molecular compositional data of
    DOM formulae that serve as input to commProc
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:8efe8f1aa43ad7261add768cdbf7590db1d3de9a7ecbf48935f4ed7d59c7d881
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - commProc
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dom-assembly-process-quantification

## Summary

Quantifies the relative influences of deterministic versus stochastic processes in the assembly of DOM assemblages using the iDOM commProc function, which computes dendrogram-based betaNTI null modeling on FT-ICR MS-derived molecular composition data.

## When to use

Use when FT-ICR MS molecular compositional data of dissolved organic matter are available and the goal is to classify pairwise turnover between assemblages into deterministic (variable/homogeneous selection) versus stochastic assembly processes via betaNTI null modeling.

## When NOT to use

- The input is not molecular compositional data of DOM and no molecular dendrogram (MCD, TD, TWCD or equivalent) of DOM formulae can be constructed, since commProc relies on dendrogram-based relatedness of formulae.
- The goal is quantifying dark matter effects on molecular interactions, DOM-microbe network specialization, or thermal response of DOM — use iDME, H2, or iCER instead of commProc.
- Only a single assemblage (no pairwise comparisons between samples or fractions) is available, so betaNTI turnover cannot be computed.

## Inputs

- Molecular compositional data of DOM assemblages from FT-ICR MS (presence/absence or relative abundance of molecular formulae across samples/fractions)
- Molecular dendrograms encoding trait-based relatedness of DOM formulae (e.g., MCD, TD, TWCD dendrograms as shipped with the iDOM example datasets)

## Outputs

- Pairwise betaNTI values for all sample/fraction comparisons per dendrogram
- Classification of each pairwise comparison into assembly categories (deterministic variable selection, homogeneous selection, stochastic/dispersal-associated processes)
- Summary table (e.g., commProc_betaNTI_results.csv) of the percentage of pairwise comparisons with betaNTI > 2 per dendrogram and per fraction, with dominance ranking of deterministic versus stochastic processes

## How to apply

Install the development version of iDOM from GitHub with devtools::install_github('jianjunwang/iDOM') in R. Load molecular dendrograms (e.g., the MCD, TD, and TWCD example dendrograms shipped with iDOM, which encode molecular trait-based relatedness of DOM formulae) and the corresponding compositional data. Execute commProc on each dendrogram to compute pairwise betaNTI values by comparing observed compositional turnover against null-model randomizations of the dendrogram. Classify each pairwise comparison using the betaNTI threshold of 2: values > 2 indicate deterministic variable selection, values < -2 indicate homogeneous selection, and |betaNTI| ≤ 2 indicates stochastic/dispersal-associated processes. Summarize, per dendrogram and per fraction, the percentage of pairwise comparisons exceeding betaNTI = 2 and rank deterministic versus stochastic process dominance; in the published case study the majority of βNTI values for MCD, TD, and TWCD exceeded 2, indicating deterministic variable selection dominated.

## Related tools

- **iDOM** (R package providing the commProc function that quantifies deterministic versus stochastic DOM assembly processes via betaNTI null modeling; ships example MCD, TD, and TWCD dendrograms) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and commProc is executed on the dendrograms and compositional data)
- **devtools** (Used to install the development version of iDOM from GitHub (devtools::install_github('jianjunwang/iDOM')))
- **FT-ICR MS** (Instrument generating the high-resolution molecular compositional data of DOM formulae that serve as input to commProc)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); library(iDOM); results <- commProc(dendrogram = MCD, comm = dom_composition); write.csv(results, 'commProc_betaNTI_results.csv')
```

## Evaluation signals

- The majority of betaNTI values for MCD, TD, and TWCD exceed 2, reproducing the Figure 4A pattern of dominant deterministic (variable selection) assembly
- Each pairwise comparison is classified using the betaNTI threshold of 2 into variable selection, homogeneous selection, or stochastic/dispersal-associated categories
- Stochastic processes are relatively more important in inactive fractions and homogeneous selection is minimal, consistent with the published case study
- A summary table is written to commProc_betaNTI_results.csv with per-dendrogram and per-fraction percentages of comparisons with betaNTI > 2
- iDOM is installed successfully via devtools::install_github('jianjunwang/iDOM') and commProc runs without errors on the example dendrograms

## Limitations

- commProc requires dendrogram-based relatedness of DOM formulae (MCD, TD, TWCD); results depend on how the molecular dendrogram is constructed and which trait metric is chosen
- The betaNTI classification is threshold-based (±2), so pairwise comparisons near the threshold have ambiguous process assignments
- Inference is relative: it ranks deterministic versus stochastic contributions per fraction rather than identifying specific selective agents

## Evidence

- [methods] commProc quantifies deterministic vs stochastic DOM assembly: "commProc: Quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages."
- [readme] iDOM installation command: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] iDOM scope statement: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
- [methods] Advanced function: assembly process quantification: "advanced analyses to quantify the assembly processes of DOM assemblages, the effect of molecular dark matter on DOM molecular interactions"
- [results] Source card finding on betaNTI dominance: "The majority of βNTI values for MCD, TD, and TWCD exceeded 2, indicating that deterministic processes, particularly variable selection, predominantly governed molecular assembly (Figure 4A)."
