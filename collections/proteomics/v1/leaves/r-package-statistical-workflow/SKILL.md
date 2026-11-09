---
name: r-package-statistical-workflow
description: Use when analyzing FT-ICR MS molecular formula abundance tables that
  include uncharacterized molecules (dark matter) and the goal is to quantify dark
  matter's effect on DOM molecular interactions through co-occurrence networks.
when_to_use_negative:
- The input is compositional FT-ICR MS data without dark matter annotations, so KK
  and DK networks cannot be contrasted
- The analysis goal is DOM assembly processes (commProc) or DOM-microbe bipartite
  network specialization (H2) rather than dark-matter effects on molecular interactions
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the iDME function that quantifies the effect of molecular
    dark matter on DOM assemblages by constructing co-occurrence networks based on
    the presence and absence of dark matter
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Execution environment in which iDOM is installed and the subsampling, network
    construction, and centrality computations are run
- name: devtools
  role: Used to install iDOM from its GitHub source within R
- name: FT-ICR MS
  role: Instrument producing the molecular compositional data (formula abundance table)
    that iDOM analyzes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:6f98e42653987e6b39402d20dcb8d15cee5181bade5496746f179d1c4584927b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# r-package-statistical-workflow

## Summary

An R-based statistical workflow for dissolved organic matter (DOM) characterized by FT-ICR MS, using the iDOM package to quantify the effect of molecular dark matter on molecular interactions via the iDME co-occurrence network function.

## When to use

Use when analyzing FT-ICR MS molecular formula abundance tables that include uncharacterized molecules (dark matter) and the goal is to quantify dark matter's effect on DOM molecular interactions through co-occurrence networks.

## When NOT to use

- The input is compositional FT-ICR MS data without dark matter annotations, so KK and DK networks cannot be contrasted
- The analysis goal is DOM assembly processes (commProc) or DOM-microbe bipartite network specialization (H2) rather than dark-matter effects on molecular interactions

## Inputs

- Molecular formula abundance table from FT-ICR MS DOM data, including dark matter (uncharacterized molecule) annotations
- Sample metadata / warming-treatment grouping (temperate vs. subtropical regions)
- Choice of network centrality metric (e.g., degree centrality)

## Outputs

- Per-replicate KK and DK co-occurrence networks with degree centrality values
- Replicate-level iDME values (relative difference in degree centrality attributable to dark matter)
- Aggregated iDME table per warming treatment (e.g., idme_results.csv) with significance tests of the temperature trend

## How to apply

Install iDOM in R via devtools::install_github('jianjunwang/iDOM') and load a molecular formula abundance table that includes dark matter annotations from the warming-experiment case study. For each of 100 subsampling replicates, draw equal numbers of known and dark-matter molecular formulas so that KK (known-only) and DK (known-plus-dark-matter) networks have the same node count. Construct KK co-occurrence networks from known formulas only and DK networks including dark matter, per the iDME design of building networks based on the presence and absence of dark matter. Compute degree centrality for each replicate's KK and DK networks and derive the iDME value as the relative difference in degree centrality attributable to dark matter; aggregate replicate-level values per warming treatment and test the temperature trend (e.g., p < 0.05 in temperate regions, nonsignificant in subtropical regions).

## Related tools

- **iDOM** (R package providing the iDME function that quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter) — https://github.com/jianjunwang/iDOM
- **R** (Execution environment in which iDOM is installed and the subsampling, network construction, and centrality computations are run)
- **devtools** (Used to install iDOM from its GitHub source within R)
- **FT-ICR MS** (Instrument producing the molecular compositional data (formula abundance table) that iDOM analyzes)

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- iDME values from degree centrality should be negative and significantly different from zero: ranging from −24.2% to −18.2% in temperate regions and from −23.0% to −21.0% in subtropical regions, matching Figure 6
- Each subsampling replicate must have equal node counts in KK and DK networks (equal numbers of known and dark-matter formulas drawn)
- The temperate temperature trend should be significant (p < 0.05) while the subtropical trend is nonsignificant
- Output table (idme_results.csv) must contain replicate-level iDME values aggregated per warming treatment
- iDOM must be installed from jianjunwang/iDOM and the iDME function must construct networks from presence/absence of dark matter

## Limitations

- iDME values depend on the chosen centrality metric and on the subsampling design; equal node counts between KK and DK networks must be enforced for a fair contrast
- The reported ranges (−24.2% to −18.2% temperate; −23.0% to −21.0% subtropical) come from the paper's example warming-experiment datasets and may not generalize to other systems

## Evidence

- [methods] iDME: Quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter.: "iDME: Quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter."
- [methods] devtools install: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] abstract dark matter handling: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [readme] README iDME description: "[iDME](https://pubs.acs.org/doi/10.1021/acs.est.2c05052): Quantifies the

effect of molecular dark matter on DOM assemblages by constructing

co-occurrence networks based on the presence and absence"
