---
name: dom-microbe-bipartite-network-construction
description: Use when you have FT-ICR MS-derived DOM molecular intensity data paired
  with microbial abundance tables and need to quantify bipartite DOM-microbe associations
  and their network-level specialization under an experimental treatment (e.g., warming).
when_to_use_negative:
- Your goal is quantifying deterministic vs. stochastic assembly of DOM assemblages
  — use the iDOM commProc function instead
- Your goal is the effect of molecular dark matter on DOM interactions — use the iDOM
  iDME function instead
- Your data lack paired DOM molecular composition (FT-ICR MS) and microbial abundance,
  so no bipartite DOM-microbe network can be constructed
edam_operation: http://edamontology.org/operation_0279
tools:
- name: iDOM
  role: Provides the H2 function that constructs DOM-microbe bipartite networks (full,
    negative, positive) and calculates network-level specialization
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which the iDOM package and its H2 function are installed
    and executed
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install the iDOM development version from GitHub (devtools::install_github)
  repo: https://github.com/jianjunwang/iDOM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:5ef12c7732a6415e7105654f1e0a572e4c77b221a25346d29ca2fbc92da34661
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0279
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dom-microbe-bipartite-network-construction

## Summary

Constructs DOM-microbe bipartite co-occurrence networks from FT-ICR MS molecular intensity data and microbial abundance tables using the iDOM H2 function, yielding full, negative, and positive network variants with network-level specialization (H2′) standardized against null models.

## When to use

Use when you have FT-ICR MS-derived DOM molecular intensity data paired with microbial abundance tables and need to quantify bipartite DOM-microbe associations and their network-level specialization under an experimental treatment (e.g., warming).

## When NOT to use

- Your goal is quantifying deterministic vs. stochastic assembly of DOM assemblages — use the iDOM commProc function instead
- Your goal is the effect of molecular dark matter on DOM interactions — use the iDOM iDME function instead
- Your data lack paired DOM molecular composition (FT-ICR MS) and microbial abundance, so no bipartite DOM-microbe network can be constructed

## Inputs

- DOM molecular intensity matrix from high-resolution FT-ICR MS (molecular compositional data)
- Microbial abundance table (e.g., 463 bacterial genera)
- Treatment/region metadata (e.g., warming vs. control, temperate vs. subtropical)

## Outputs

- DOM-microbe bipartite co-occurrence networks (full, negative, and positive variants)
- Edge counts per network variant (e.g., 1108 negative and 1938 positive interactions)
- Null-model-standardized specialization values (H2′) with significance (p < 0.05)
- Per-treatment/per-region H2′ comparison results (e.g., h2_network_specialization_results.csv)

## How to apply

Install the iDOM R package from GitHub with devtools::install_github('jianjunwang/iDOM'), then load the molecular intensity matrix and the microbial abundance table (e.g., 463 bacterial genera). Call the iDOM H2 function to construct DOM-microbe bipartite co-occurrence networks and derive the full, negative, and positive network variants. Standardize H2 against null models to obtain H2′ values, assessing significance at p < 0.05. Check the expected network composition against known edge counts (e.g., 1108 negative and 1938 positive DOM-bacteria interactions in the example warming dataset) before comparing H2′ between treatments (warming vs. control) within each region.

## Related tools

- **iDOM** (Provides the H2 function that constructs DOM-microbe bipartite networks (full, negative, positive) and calculates network-level specialization) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which the iDOM package and its H2 function are installed and executed) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the iDOM development version from GitHub (devtools::install_github)) — https://github.com/jianjunwang/iDOM

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Output reports the expected edge counts for the example dataset: 1108 negative and 1938 positive DOM-bacteria interactions
- Standardized H2′ values are negative and significantly lower than expected by chance (p < 0.05)
- H2′ comparison reproduces the region-specific warming response: positive-network H2′ decreases in both regions, negative-network H2′ increases only in the temperate region, no significant change in the subtropical negative networks (consistent with Figure 7)
- All three network variants (full, negative, positive) are produced, matching the H2 function's documented scope
- Results file (h2_network_specialization_results.csv) contains edge counts, H2′ values, and per-region warming response summaries

## Limitations

- Standardization against null models is required to interpret H2′; raw H2 values are not directly comparable
- Warming effects are region-specific: negative-network H2′ increased only in the temperate region and showed no significant correlation in the subtropical region, so results may not generalize across regions
- Interpretation depends on splitting networks into negative and positive association variants; sign conventions of co-occurrence edges must be preserved

## Evidence

- [methods] H2 function constructs DOM-microbe bipartite networks including full, negative and positive networks: "H2: Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks"
- [readme] README on H2 covering full, negative and positive networks: "Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks, including full, negative and positive networks."
- [methods] Installation from GitHub with devtools: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] Abstract describing the DOM-microbe network and warming case study: "analyze the ecological networks between DOM and microbes, and explore their response to warming"
