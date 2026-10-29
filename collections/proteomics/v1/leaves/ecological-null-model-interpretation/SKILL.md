---
name: ecological-null-model-interpretation
description: Use when FT-ICR MS-derived DOM compositional data and molecular trait
  dendrograms (e.g. MCD, TD, TWCD) are available and the goal is to classify pairwise
  turnover into variable selection, homogeneous selection, or stochastic processes
  via betaNTI.
when_to_use_negative:
- Input lacks molecular trait information or a dendrogram relating DOM formulae, since
  betaNTI is dendrogram-based rather than phylogeny-based
- The goal is dark-matter effects on molecular interactions (use iDME) or DOM-microbe
  network specialization (use H2), not assembly-process quantification
- Only a single sample/community is available — betaNTI requires pairwise comparisons
  between assemblages
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the commProc function that quantifies deterministic vs
    stochastic assembly processes of DOM assemblages via betaNTI null modeling
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM/commProc is installed and executed
- name: devtools
  role: Used to install the development version of iDOM from GitHub
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:2781799824ec3664ce82eb1a6ec5d86ed3cd44913dc09f46059ed39367c4098a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - commProc
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ecological-null-model-interpretation

## Summary

Quantifies the relative influences of deterministic and stochastic processes in DOM assemblage assembly using dendrogram-based betaNTI null modeling implemented in the commProc function of the iDOM R package, applied to FT-ICR MS molecular data.

## When to use

Use when FT-ICR MS-derived DOM compositional data and molecular trait dendrograms (e.g. MCD, TD, TWCD) are available and the goal is to classify pairwise turnover into variable selection, homogeneous selection, or stochastic processes via betaNTI.

## When NOT to use

- Input lacks molecular trait information or a dendrogram relating DOM formulae, since betaNTI is dendrogram-based rather than phylogeny-based
- The goal is dark-matter effects on molecular interactions (use iDME) or DOM-microbe network specialization (use H2), not assembly-process quantification
- Only a single sample/community is available — betaNTI requires pairwise comparisons between assemblages

## Inputs

- Molecular dendrograms (e.g. MCD, TD, TWCD) encoding trait-based relatedness of DOM formulae
- DOM molecular compositional data from FT-ICR MS (presence/absence or relative abundance of formulae)
- Fraction or sample-group metadata for stratifying pairwise comparisons

## Outputs

- Pairwise betaNTI values for each dendrogram (MCD, TD, TWCD) and fraction
- Per-comparison assembly-process classification (variable selection, homogeneous selection, stochastic)
- Summary of percentage of betaNTI values > 2 and deterministic-vs-stochastic dominance ranking, written to commProc_betaNTI_results.csv

## How to apply

Install the development version of iDOM from GitHub with devtools::install_github('jianjunwang/iDOM'). Load the molecular dendrograms (MCD, TD, TWCD) shipped with iDOM as example datasets, which encode molecular trait-based relatedness of DOM formulae. Run commProc on each dendrogram to compute pairwise betaNTI values by comparing observed compositional turnover against null-model randomizations of the dendrogram. Classify each pairwise comparison using the betaNTI threshold of 2: values > 2 indicate deterministic variable selection, values < -2 indicate homogeneous selection, and values between -2 and 2 indicate stochastic/dispersal-associated processes. Summarize, per dendrogram and per fraction, the percentage of pairwise comparisons with betaNTI exceeding 2 and rank deterministic versus stochastic dominance.

## Related tools

- **iDOM** (R package providing the commProc function that quantifies deterministic vs stochastic assembly processes of DOM assemblages via betaNTI null modeling) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM/commProc is installed and executed)
- **devtools** (Used to install the development version of iDOM from GitHub)

## Examples

```
# install.packages("devtools"); devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Majority of betaNTI values for MCD, TD, and TWCD exceed 2, reproducing the Figure 4A pattern of dominant deterministic (variable selection) assembly
- Stochastic processes are relatively more important in inactive fractions, and homogeneous selection (betaNTI < -2) is minimal
- betaNTI classifications consistently fall into the three standard categories using the threshold of 2
- Summary results are written to commProc_betaNTI_results.csv with per-dendrogram and per-fraction percentages

## Limitations

- betaNTI depends on the chosen molecular dendrogram (MCD, TD, TWCD); results may differ across dendrogram types
- Interpretation of the |betaNTI| > 2 threshold as deterministic is a convention from null-model ecology and relies on adequate null randomizations
- Results are interpreted at the level of pairwise comparison fractions; conclusions require aggregation across many pairs

## Evidence

- [methods] commProc function description: "commProc: Quantifies the relative influences of deterministic and stochastic processes in the assembly of DOM assemblages."
- [readme] iDOM installation: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] package scope for assembly quantification: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions"
- [readme] package availability: "The source code and example dataset of iDOM are publicly available on https://github.com/jianjunwang/iDOM"
