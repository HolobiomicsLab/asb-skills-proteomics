---
name: network-level-specialization-calculation
description: Use when you have a DOM molecular intensity matrix (from FT-ICR MS) and
  a microbial abundance table (e.
when_to_use_negative:
- Input is only a single-trophic-level molecular or abundance table with no paired
  DOM and microbial data, so no bipartite DOM-microbe network can be constructed
- The goal is to quantify DOM assembly processes or dark-matter effects on molecular
  interactions — use the iDOM commProc or iDME functions instead of H2
- You need a univariate compositional environmental response (thermal response of
  DOM) — the iCER function is the appropriate iDOM tool, not H2
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: iDOM
  role: Provides the H2 function that calculates network-level specialization of all
    interacting trophic levels in DOM-microbe bipartite networks, including full,
    negative and positive networks
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which the iDOM package and its H2 function are installed
    and executed (via devtools::install_github)
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install the iDOM development version from GitHub before running H2
  repo: https://github.com/jianjunwang/iDOM
- name: FT-ICR MS
  role: Instrument that produces the DOM molecular composition data (molecular intensity
    matrix) serving as one bipartite-network input layer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:b82c45de847451c2c8678b55c499bbe1df0c3265f50ff0e5b94d31c483ca91f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# network-level-specialization-calculation

## Summary

Calculates the network-level specialization (H2) of all interacting trophic levels in DOM-microbe bipartite networks — including full, negative, and positive network variants — using the H2 function of the iDOM R package, and standardizes it against null models to test whether warming alters specialization.

## When to use

Use when you have a DOM molecular intensity matrix (from FT-ICR MS) and a microbial abundance table (e.g., 463 bacterial genera) and need network-level specialization (H2′) of bipartite DOM-microbe co-occurrence networks, or when comparing specialization between environmental treatments such as warming versus control.

## When NOT to use

- Input is only a single-trophic-level molecular or abundance table with no paired DOM and microbial data, so no bipartite DOM-microbe network can be constructed
- The goal is to quantify DOM assembly processes or dark-matter effects on molecular interactions — use the iDOM commProc or iDME functions instead of H2
- You need a univariate compositional environmental response (thermal response of DOM) — the iCER function is the appropriate iDOM tool, not H2

## Inputs

- DOM molecular intensity matrix from high-resolution (FT-ICR MS) mass spectrometry
- Microbial abundance table (e.g., 463 bacterial genera)
- Treatment/region metadata (warming vs. control; temperate vs. subtropical)

## Outputs

- Full, negative, and positive DOM-microbe bipartite network variants with edge counts (e.g., 1108 negative and 1938 positive DOM-bacteria interactions)
- Network-level specialization values (H2) and null-model-standardized H2′ with significance (p < 0.05)
- Per-region, per-network-sign warming response summary (e.g., h2_network_specialization_results.csv)

## How to apply

Install iDOM from GitHub with devtools::install_github("jianjunwang/iDOM") and load the FT-ICR MS-derived molecular intensity matrix together with the microbial abundance table (463 bacterial genera). Call the iDOM H2 function to construct DOM-microbe bipartite co-occurrence networks and derive the full, negative, and positive network variants; the example datasets yield 1108 negative and 1938 positive DOM-bacteria interactions. Standardize H2 against null models to obtain H2′ values, assessing significance at p < 0.05. Decision point: analyze full, negative, and positive networks separately because warming responses differ by sign — positive-network H2′ decreases under warming in both temperate and subtropical regions, negative-network H2′ increases only in the temperate region, and the subtropical negative networks show no significant correlation. Write edge counts, H2′ values, and per-region warming response summaries to a results CSV (e.g., h2_network_specialization_results.csv).

## Related tools

- **iDOM** (Provides the H2 function that calculates network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks, including full, negative and positive networks) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which the iDOM package and its H2 function are installed and executed (via devtools::install_github)) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the iDOM development version from GitHub before running H2) — https://github.com/jianjunwang/iDOM
- **FT-ICR MS** (Instrument that produces the DOM molecular composition data (molecular intensity matrix) serving as one bipartite-network input layer)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); library(iDOM); H2(dom_intensity_matrix, microbe_abundance_table)
```

## Evaluation signals

- Expected network composition reproduced: 1108 negative and 1938 positive DOM-bacteria interactions in the example DOM-microbe datasets
- H2′ values are standardized against null models and are negative and significantly lower than expected by chance (p < 0.05)
- Warming response pattern matches expectations: positive-network H2′ decreases in both temperate and subtropical regions; negative-network H2′ increases only in the temperate region; subtropical negative networks show no significant correlation
- Output file h2_network_specialization_results.csv contains edge counts, H2′ values, and per-region warming response summary consistent with Figure 7
- All three network variants (full, negative, positive) are reported, not just the full network

## Limitations

- H2 is specific to bipartite DOM-microbe networks; it addresses network-level specialization of interacting trophic levels and does not quantify DOM assembly processes (commProc) or dark-matter effects (iDME)
- Interpretation of warming responses depends on region and network sign: negative-network H2′ responded to warming only in the temperate region, and subtropical negative networks showed no significant correlation, so results are not transferable across regions without re-standardization
- iDOM is a development version installed from GitHub (zip or tar.gz source), which may require installing from local source with repos = NULL and type = 'source'

## Evidence

- [methods] H2 function computes network-level specialization including full, negative and positive networks: "H2: Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks"
- [readme] README describes H2 covering full, negative and positive networks: "Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks, including full, negative and positive networks."
- [abstract] Article abstract motivates DOM-microbe network analysis under warming: "analyze the ecological networks between DOM and microbes, and explore their response to warming"
- [readme] README installation command for iDOM: "devtools::install_github("jianjunwang/iDOM")"
