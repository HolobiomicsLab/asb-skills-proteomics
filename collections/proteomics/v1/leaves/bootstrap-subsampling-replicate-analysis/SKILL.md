---
name: bootstrap-subsampling-replicate-analysis
description: Use when comparing co-occurrence network statistics between known-formula
  (KK) and dark-matter-inclusive (DK) networks from DOM molecular formula tables,
  since network metrics like degree centrality depend on node count and subsampling
  removes that confound.
when_to_use_negative:
- When the KK and DK networks already have identical node counts by construction (subsampling
  only adds unnecessary randomness)
- When the goal is quantifying deterministic vs. stochastic DOM assembly processes
  (use the commProc function instead) or DOM-microbe network specialization (use H2)
- When molecular formula annotations distinguishing known molecules from dark matter
  are unavailable — the KK/DK split cannot be constructed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the iDME function that constructs KK/DK co-occurrence
    networks based on the presence and absence of dark matter and quantifies the dark-matter
    effect on DOM assemblages
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which the iDOM package, subsampling replicates, and
    iDME computation are executed
- name: devtools
  role: Used to install iDOM from GitHub for running the subsampling/iDME workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:f276fd4ea59580ed095ad052a424d9d3c2453b55a25a9c81835c10d509d2de5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# bootstrap-subsampling-replicate-analysis

## Summary

A bootstrap-style subsampling scheme that draws many (e.g., 100) replicate pairs of equal-sized formula sets from FT-ICR MS molecular formula tables so that co-occurrence networks built with (DK) and without (KK) molecular dark matter are node-count matched, enabling a fair iDME estimate of dark matter's effect on degree centrality.

## When to use

Use when comparing co-occurrence network statistics between known-formula (KK) and dark-matter-inclusive (DK) networks from DOM molecular formula tables, since network metrics like degree centrality depend on node count and subsampling removes that confound.

## When NOT to use

- When the KK and DK networks already have identical node counts by construction (subsampling only adds unnecessary randomness)
- When the goal is quantifying deterministic vs. stochastic DOM assembly processes (use the commProc function instead) or DOM-microbe network specialization (use H2)
- When molecular formula annotations distinguishing known molecules from dark matter are unavailable — the KK/DK split cannot be constructed

## Inputs

- Molecular formula abundance table from FT-ICR MS (e.g., the iDOM warming-experiment dataset) with annotations distinguishing known molecules from dark matter
- Treatment/group labels (e.g., warming treatment, temperate vs. subtropical region) for aggregating replicate-level results
- Number of subsampling replicates (e.g., 100) and number of formulas per draw for each of KK and DK

## Outputs

- Replicate-level iDME values (relative difference in degree centrality between DK and KK networks)
- Aggregated iDME statistics per treatment with significance tests (e.g., temperate range -24.2% to -18.2%; subtropical range -23.0% to -21.0%)
- Results table (e.g., idme_results.csv) of replicate-level iDME values

## How to apply

From a molecular formula abundance table (e.g., the iDOM warming-experiment dataset with dark-matter annotations), run 100 subsampling replicates, each drawing equal numbers of known and dark-matter molecular formulas so the KK and DK networks have the same node count. For each replicate, construct the KK co-occurrence network from known formulas only and the DK network including dark matter, following the iDME design of building networks based on the presence and absence of dark matter. Compute degree centrality for each replicate's KK and DK networks and derive the iDME value as the relative difference in degree centrality attributable to dark matter. Aggregate replicate-level iDME values per treatment (here, warming treatment) and test trends statistically (e.g., temperate sites: significant increase, p < 0.05; subtropical: nonsignificant). The equal-node-count subsampling is the key design decision: it prevents the trivially higher connectivity of larger networks from masquerading as a dark-matter effect.

## Related tools

- **iDOM** (R package providing the iDME function that constructs KK/DK co-occurrence networks based on the presence and absence of dark matter and quantifies the dark-matter effect on DOM assemblages) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which the iDOM package, subsampling replicates, and iDME computation are executed)
- **devtools** (Used to install iDOM from GitHub for running the subsampling/iDME workflow)

## Examples

```
install.packages("devtools"); devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Each subsampling replicate yields KK and DK networks with the same node count (equal numbers of known and dark-matter formulas drawn)
- iDME values computed from degree centrality are negative and significantly different from zero, matching the published ranges: -24.2% to -18.2% (temperate) and -23.0% to -21.0% (subtropical)
- Replicate-level iDME values aggregate to ~100 values per warming treatment, with temperate showing a significant increase (p < 0.05) and subtropical nonsignificant
- Results table (idme_results.csv) is written with one iDME value per replicate per treatment
- Values match the dark-matter effect ranges reported in Figure 6 of the iDOM paper

## Limitations

- The comparison depends on iDOM's iDME network-construction design (networks built on presence/absence of dark matter); results are specific to co-occurrence networks and degree centrality as the network metric
- Replicate-level iDME significance (temperate significant, subtropical nonsignificant) reflects the example warming-experiment datasets and may not generalize to other systems
- Subsampling discards part of the data each replicate; conclusions rely on aggregation across the 100 replicates rather than the full network

## Evidence

- [readme] iDME: Quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter.: "iDME: Quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter."
- [methods] devtools install from GitHub: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] package handles dark matter data type: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [abstract] case study on dark matter effects on molecular interactions: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions"
