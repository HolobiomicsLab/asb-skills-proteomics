---
name: positive-negative-interaction-partitioning
description: Use when FT-ICR MS molecular intensity data and microbial abundance tables
  are available and the goal is to test how positive vs. negative DOM-microbe interactions
  respond to an environmental treatment such as warming.
when_to_use_negative:
- Input lacks paired DOM molecular and microbial taxon abundance data, so a bipartite
  DOM-microbe network cannot be constructed
- The question is DOM assemblage assembly or dark-matter effects rather than interaction
  specialization — use the commProc or iDME functions instead
- Only univariate DOM thermal response is needed — use the iCER indicator of compositional-level
  environmental response rather than bipartite network partitioning
tools:
- name: iDOM
  role: R package providing the H2 function that constructs DOM-microbe bipartite
    co-occurrence networks and computes H2 specialization for full, negative, and
    positive network variants
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and the H2 specialization analysis
    is executed
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install iDOM from GitHub via devtools::install_github("jianjunwang/iDOM")
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:3ac1d8bc7e52132b4a611e3066b62999c0e60a58e42716c82945170d46718e43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# positive-negative-interaction-partitioning

## Summary

Partitions DOM-microbe bipartite co-occurrence networks into full, negative, and positive interaction networks and quantifies network-level standardized specialization (H2′) of each variant using the iDOM H2 function, so warming responses can be compared between cooperative and competitive interaction pools.

## When to use

Use when FT-ICR MS molecular intensity data and microbial abundance tables are available and the goal is to test how positive vs. negative DOM-microbe interactions respond to an environmental treatment such as warming.

## When NOT to use

- Input lacks paired DOM molecular and microbial taxon abundance data, so a bipartite DOM-microbe network cannot be constructed
- The question is DOM assemblage assembly or dark-matter effects rather than interaction specialization — use the commProc or iDME functions instead
- Only univariate DOM thermal response is needed — use the iCER indicator of compositional-level environmental response rather than bipartite network partitioning

## Inputs

- DOM molecular intensity matrix from high-resolution FT-ICR MS (molecular compositional data)
- Microbial abundance table (e.g., 463 bacterial genera)
- Treatment/region metadata (warming vs. control, temperate vs. subtropical)

## Outputs

- Full, negative, and positive DOM-microbe bipartite network edge lists (e.g., 1108 negative and 1938 positive interactions)
- Network-level H2 and null-standardized H2′ values per network variant with significance (p < 0.05)
- Per-region warming response summary (e.g., h2_network_specialization_results.csv)

## How to apply

Install the iDOM R package from GitHub (devtools::install_github("jianjunwang/iDOM")) and load the molecular intensity matrix together with the microbial abundance table (e.g., 463 bacterial genera in the case study). Run the iDOM H2 function, which constructs DOM-microbe bipartite co-occurrence networks and computes network-level specialization for the full network as well as its negative and positive variants. Standardize each H2 against null models to obtain H2′ values, assessing significance at p < 0.05. Record the edge counts per sign (the reference case study yields 1108 negative and 1938 positive DOM-bacteria interactions) and compare H2′ between warming and control treatments per region: positive-network H2′ decreases under warming in both temperate and subtropical regions, negative-network H2′ increases only in the temperate region, and the subtropical negative network shows no significant response. Write edge counts, H2′ values, and per-region warming response summaries to an output table such as h2_network_specialization_results.csv.

## Related tools

- **iDOM** (R package providing the H2 function that constructs DOM-microbe bipartite co-occurrence networks and computes H2 specialization for full, negative, and positive network variants) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and the H2 specialization analysis is executed) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install iDOM from GitHub via devtools::install_github("jianjunwang/iDOM"))

## Examples

```
devtools::install_github("jianjunwang/iDOM"); library(iDOM); H2(dom_intensity_matrix, microbe_abundance_table)
```

## Evaluation signals

- Output edge counts match the reference composition: 1108 negative and 1938 positive DOM-bacteria interactions
- Standardized H2′ values are negative and significantly lower than null-model expectation (p < 0.05)
- Positive-network H2′ decreases under warming in both temperate and subtropical regions
- Negative-network H2′ increases under warming only in the temperate region, with no significant correlation in the subtropical region
- Results are consistent with Figure 7 of the underlying warming study when reproduced

## Limitations

- Significance of H2′ depends on null-model standardization; results are only interpretable relative to the chance expectation (p < 0.05)
- Warming responses are region-specific, so conclusions from one region (e.g., subtropical negative networks showing no significant response) do not generalize
- The H2 function targets DOM-microbe bipartite networks from FT-ICR MS molecular data; it is not a general-purpose unipartite network specialization tool

## Evidence

- [methods] H2 function description in methods filter steps: "H2: Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks"
- [readme] README key-function description including network sign variants: "Calculates the
network-level specialization of all interacting trophic levels in
DOM-microbe bipartite networks, including full, negative and positive
networks."
- [abstract] Abstract framing of DOM-microbe networks and warming response: "analyze the ecological networks between DOM and microbes, and explore their response to warming"
- [readme] README installation command for the R package: "devtools::install_github("jianjunwang/iDOM")"
