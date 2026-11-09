---
name: dark-matter-cooccurrence-network-construction
description: Use when an FT-ICR MS molecular formula table contains both characterized
  formulas and dark matter (unannotated peaks) and the goal is to quantify the effect
  of dark matter on DOM molecular interactions, e.g. comparing temperate vs. subtropical
  warming treatments.
when_to_use_negative:
- The dataset contains only fully characterized molecular formulas with no dark matter
  annotations — the KK/DK contrast cannot be built
- The goal is quantifying deterministic vs. stochastic DOM assembly processes rather
  than dark-matter effects on molecular interactions (use commProc instead)
- The analysis targets DOM-microbe bipartite network specialization (use the H2 function)
  or thermal response (iCER) rather than dark-matter effects
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3339
tools:
- name: iDOM
  role: R package supplying the iDME function that constructs co-occurrence networks
    based on the presence and absence of dark matter to quantify dark-matter effects
    on DOM assemblages
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment for installing iDOM and running the iDME subsampling/network
    workflow
- name: devtools
  role: Used to install iDOM from its GitHub source before running iDME
- name: FT-ICR MS
  role: Instrument producing the molecular formula composition data (including dark
    matter peaks) that iDME analyzes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:a65f656c67aee118789c96c0c3b5b8221399904695a9d58a1139b84d68fe234e
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
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dark-matter-cooccurrence-network-construction

## Summary

Construction of paired co-occurrence networks with and without uncharacterized (dark matter) molecular formulas to quantify how unannotated FT-ICR MS peaks alter DOM molecular interactions, via the iDME function in the iDOM R package.

## When to use

Use when an FT-ICR MS molecular formula table contains both characterized formulas and dark matter (unannotated peaks) and the goal is to quantify the effect of dark matter on DOM molecular interactions, e.g. comparing temperate vs. subtropical warming treatments.

## When NOT to use

- The dataset contains only fully characterized molecular formulas with no dark matter annotations — the KK/DK contrast cannot be built
- The goal is quantifying deterministic vs. stochastic DOM assembly processes rather than dark-matter effects on molecular interactions (use commProc instead)
- The analysis targets DOM-microbe bipartite network specialization (use the H2 function) or thermal response (iCER) rather than dark-matter effects

## Inputs

- Molecular formula abundance table from FT-ICR MS of DOM samples, with dark matter (uncharacterized molecule) annotations
- Warming-experiment sample metadata (e.g. temperate vs. subtropical treatments) for grouping replicates

## Outputs

- Per-replicate iDME values (relative difference in degree centrality between DK and KK networks)
- Paired KK (known-only) and DK (dark-matter-inclusive) co-occurrence networks with equal node counts
- Aggregated iDME table per treatment (e.g. idme_results.csv) with temperature-trend significance tests

## How to apply

In R, install the iDOM package from GitHub via devtools::install_github('jianjunwang/iDOM') and load the warming-experiment molecular formula abundance table that includes dark matter annotations. For each of ~100 subsampling replicates, draw equal numbers of known (KK) and dark-matter-inclusive (DK) molecular formulas so both networks have the same node count — this equal-node subsampling is the key design decision that isolates the dark-matter effect from network size. Construct KK co-occurrence networks from known formulas only and DK networks that include dark matter, following the iDME design of building networks based on the presence and absence of dark matter. Compute degree centrality for each replicate's KK and DK networks and derive iDME as the relative difference in degree centrality attributable to dark matter; aggregate replicate-level iDME values per warming treatment and test the temperature trend (significant increase p < 0.05 in temperate, nonsignificant in subtropical). Correct application reproduces the reference ranges: iDME from degree centrality of −24.2% to −18.2% in temperate and −23.0% to −21.0% in subtropical regions.

## Related tools

- **iDOM** (R package supplying the iDME function that constructs co-occurrence networks based on the presence and absence of dark matter to quantify dark-matter effects on DOM assemblages) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment for installing iDOM and running the iDME subsampling/network workflow)
- **devtools** (Used to install iDOM from its GitHub source before running iDME)
- **FT-ICR MS** (Instrument producing the molecular formula composition data (including dark matter peaks) that iDME analyzes)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); res <- iDME(formula_table = dom_abundances, dark = dark_matter_annot, nrep = 100)
```

## Evaluation signals

- Computed temperate iDME values from degree centrality fall within −24.2% to −18.2% and subtropical values within −23.0% to −21.0%, matching Figure 6
- KK and DK networks are built from equal numbers of molecular formulas per replicate (same node count), so iDME is not confounded by network size
- All iDME values are negative and significantly different from zero, consistent with dark matter enhancing molecular interactions
- Replicate-level iDME values are aggregated per warming treatment and the temperate trend is significant (p < 0.05) while the subtropical trend is nonsignificant
- Results are written to the expected output file idme_results.csv

## Limitations

- Dark matter (uncharacterized FT-ICR MS peaks) cannot be assigned molecular classes or traits, so the effect is quantified only at the interaction (degree centrality) level
- Estimates depend on subsampling: ~100 replicates of equal-node known/dark-matter draws are needed, so results vary with replicate count
- The reported iDME ranges were demonstrated only on the example experimental warming dataset (temperate and subtropical regions) and may not generalize to other systems

## Evidence

- [methods] iDME: Quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter.: "iDME: Quantifies the effect of molecular dark matter on DOM assemblages by constructing co-occurrence networks based on the presence and absence of dark matter."
- [abstract] The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter).: "uncharacterized molecules (i.e., dark matter)"
- [readme] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [readme] Dark Matter Enhances Interactions within Both Microbes and Dissolved Organic Matter under Global Change: "Dark Matter Enhances Interactions within Both Microbes and Dissolved Organic Matter under Global Change"
