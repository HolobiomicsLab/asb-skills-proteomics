---
name: warming-response-network-comparison
description: Use when paired DOM molecular intensity (FT-ICR MS) and microbial abundance
  tables are available under warming vs control treatments in multiple regions, and
  the goal is testing whether bipartite network specialization (H2′) responds to warming.
when_to_use_negative:
- The goal is quantifying deterministic vs stochastic assembly of DOM assemblages
  — use the iDOM commProc function instead
- The goal is quantifying the effect of molecular dark matter on DOM molecular interactions
  — use the iDOM iDME function instead
- The data lack a warming/control treatment contrast or paired DOM and microbial abundance
  tables, so no H2′ treatment comparison is possible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: Provides the H2 function that calculates network-level specialization of full,
    negative, and positive DOM-microbe bipartite networks under warming
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which the iDOM package and its H2 function are executed
- name: devtools
  role: Used to install iDOM from GitHub before running the H2 network analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:fc4fac8ea9c0c0df3095219c8b532ddada964e72d81b7c3a75c7cb3bcf3b9547
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
yes_rate: 0.6363636363636364
n_claims: 11
---

# warming-response-network-comparison

## Summary

Compares standardized network-level specialization (H2′) of DOM-microbe bipartite networks between experimental warming and control treatments, separately for negative and positive networks, to detect region-specific thermal responses of DOM-microbe associations.

## When to use

Use when paired DOM molecular intensity (FT-ICR MS) and microbial abundance tables are available under warming vs control treatments in multiple regions, and the goal is testing whether bipartite network specialization (H2′) responds to warming.

## When NOT to use

- The goal is quantifying deterministic vs stochastic assembly of DOM assemblages — use the iDOM commProc function instead
- The goal is quantifying the effect of molecular dark matter on DOM molecular interactions — use the iDOM iDME function instead
- The data lack a warming/control treatment contrast or paired DOM and microbial abundance tables, so no H2′ treatment comparison is possible

## Inputs

- DOM molecular intensity matrix from FT-ICR MS characterization
- Microbial abundance table (e.g., 463 bacterial genera)
- Treatment/region metadata assigning samples to warming vs control and temperate vs subtropical regions

## Outputs

- H2′ (null-standardized specialization) values for full, negative, and positive DOM-microbe bipartite networks per treatment and region
- Significance assessments against null models (p < 0.05)
- Edge counts of negative and positive DOM-bacteria interactions
- Per-region warming response summary (e.g., h2_network_specialization_results.csv)

## How to apply

Install the iDOM R package from GitHub (devtools::install_github("jianjunwang/iDOM")) and load the molecular intensity matrix and the microbial abundance table (e.g., 463 bacterial genera). Use the iDOM H2 function, which calculates network-level specialization for all interacting trophic levels in DOM-microbe bipartite networks, to build full, negative, and positive network variants. Standardize H2 against null models to obtain H2′ with significance assessed at p < 0.05. Check network composition against expected edge counts (1108 negative and 1938 positive DOM-bacteria interactions in the example datasets). Then compare H2′ between warming and control treatments per region: the expected pattern is that positive-network H2′ decreases under warming in both temperate and subtropical regions, while negative-network H2′ increases only in the temperate region and shows no significant change in the subtropical region. Write edge counts, H2′ values, and the per-region warming response summary to a results file (e.g., h2_network_specialization_results.csv).

## Related tools

- **iDOM** (Provides the H2 function that calculates network-level specialization of full, negative, and positive DOM-microbe bipartite networks under warming) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which the iDOM package and its H2 function are executed)
- **devtools** (Used to install iDOM from GitHub before running the H2 network analysis)

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Output edge counts match the expected network composition: 1108 negative and 1938 positive DOM-bacteria interactions
- Standardized H2′ values are negative and significantly lower than expected by chance (p < 0.05)
- Positive-network H2′ decreases under warming in both temperate and subtropical regions
- Negative-network H2′ increases under warming only in the temperate region and shows no significant change in the subtropical region
- Results file (h2_network_specialization_results.csv) contains edge counts, H2′ values, and per-region warming response summaries consistent with Figure 7

## Limitations

- Warming responses are region-specific: negative-network H2′ responds significantly only in the temperate region and shows no significant correlation in the subtropical region, so a single pooled analysis can mask region-specific effects
- H2′ interpretation depends on null-model standardization; raw H2 values are not directly comparable across networks of different sizes
- Requires paired DOM molecular (FT-ICR MS) and microbial abundance data; uncharacterized molecules (dark matter) can affect molecular interaction networks and should be considered

## Evidence

- [methods] H2 function abstracts specialization metric: "H2: Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks"
- [readme] README describes full/negative/positive variants: "Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks, including full, negative and positive networks."
- [abstract] Abstract names warming response of DOM-microbe networks as an iDOM capability: "analyze the ecological networks between DOM and microbes, and explore their response to warming"
- [readme] README gives installation command for iDOM: "devtools::install_github("jianjunwang/iDOM")"
