---
name: null-model-significance-testing
description: Use when you have a DOM-microbe bipartite co-occurrence network (full,
  negative, or positive variant) and need significance testing of network-level specialization
  beyond the raw H2 value.
when_to_use_negative:
- Input is a single unipartite network or non-bipartite interaction matrix — the H2/H2'
  metric in iDOM is defined for DOM-microbe bipartite networks.
- The goal is quantifying DOM assembly processes or dark-matter effects — use iDOM's
  commProc or iDME functions instead of H2 null-model standardization.
- Only a raw H2 value is needed for descriptive reporting and no chance-expectation
  comparison is required.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: iDOM
  role: R package providing the H2 function that calculates network-level specialization
    of full, negative, and positive DOM-microbe bipartite networks, which is then
    standardized against null models
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: runtime environment in which iDOM is installed (via devtools) and the null-model
    standardization is executed
- name: devtools
  role: used to install iDOM from GitHub (devtools::install_github('jianjunwang/iDOM'))
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:b9bee15e648d45bef35a4cbb70d87d62015b2e19a0143f1d341e3ec7fa030feb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# null-model-significance-testing

## Summary

Standardizes an observed network-level specialization metric (H2) against null models to obtain H2' and assess whether DOM-microbe bipartite network specialization is significantly lower or higher than expected by chance (p < 0.05).

## When to use

Use when you have a DOM-microbe bipartite co-occurrence network (full, negative, or positive variant) and need significance testing of network-level specialization beyond the raw H2 value.

## When NOT to use

- Input is a single unipartite network or non-bipartite interaction matrix — the H2/H2' metric in iDOM is defined for DOM-microbe bipartite networks.
- The goal is quantifying DOM assembly processes or dark-matter effects — use iDOM's commProc or iDME functions instead of H2 null-model standardization.
- Only a raw H2 value is needed for descriptive reporting and no chance-expectation comparison is required.

## Inputs

- DOM molecular intensity matrix from FT-ICR MS characterization
- microbial abundance table (e.g., 463 bacterial genera)
- bipartite network or interaction matrix (full, negative, and positive variants)
- treatment/region grouping labels (e.g., warming vs. control, temperate vs. subtropical)

## Outputs

- H2' (standardized network-level specialization) per network variant with significance p-values
- edge counts per network sign (e.g., 1108 negative and 1938 positive DOM-bacteria interactions)
- per-treatment comparison summary (e.g., written to h2_network_specialization_results.csv)

## How to apply

Install the iDOM R package from GitHub (devtools::install_github('jianjunwang/iDOM')) and load the DOM molecular intensity matrix (FT-ICR MS) together with the microbial abundance table (e.g., 463 bacterial genera). Use the iDOM H2 function to compute network-level specialization for the full, negative, and positive networks separately, since warming effects can diverge between interaction signs. Standardize each observed H2 against null models that randomize the bipartite interactions to obtain H2' and a p-value, with significance judged at p < 0.05. Interpret negative H2' values as specialization significantly lower than expected by chance. Finally, compare H2' between treatment groups (e.g., warming vs. control per region) to detect region- and sign-specific responses, and write edge counts and H2' values to an output CSV.

## Related tools

- **iDOM** (R package providing the H2 function that calculates network-level specialization of full, negative, and positive DOM-microbe bipartite networks, which is then standardized against null models) — https://github.com/jianjunwang/iDOM
- **R** (runtime environment in which iDOM is installed (via devtools) and the null-model standardization is executed)
- **devtools** (used to install iDOM from GitHub (devtools::install_github('jianjunwang/iDOM')))

## Examples

```
install.packages("devtools"); devtools::install_github("jianjunwang/iDOM"); library(iDOM); H2(net)
```

## Evaluation signals

- Output edge counts match expected composition: 1108 negative and 1938 positive DOM-bacteria interactions in the example datasets.
- Standardized H2' values are negative and significantly lower than expected by chance (p < 0.05) for the example DOM-microbe datasets.
- Positive-network H2' decreases under warming in both temperate and subtropical regions, while negative-network H2' increases only in the temperate region.
- No significant warming correlation is observed for negative networks in the subtropical region, matching Figure 7 of the associated work.
- Results table (h2_network_specialization_results.csv) contains edge counts, H2' values, and per-region warming response summaries for full, negative, and positive networks.

## Limitations

- H2' significance depends on the null model used for randomization; results are reported relative to chance expectations (p < 0.05 threshold) rather than as absolute specialization values.
- Warming responses are sign- and region-specific — an increase in negative-network H2' in one region (temperate) may have no counterpart (no significant correlation) in another (subtropical), so conclusions should not be generalized across regions or interaction signs.
- iDOM must be installed from GitHub via devtools (or from a local zip/tar.gz with repos = NULL, type = 'source'), so a working R build toolchain is required.

## Evidence

- [readme] H2 function description: "H2: Calculates the network-level specialization of all interacting trophic levels in DOM-microbe bipartite networks, including full, negative and positive networks."
- [readme] installation from GitHub: "devtools::install_github("jianjunwang/iDOM")"
- [methods] H2 listed among candidate fine-grained sub-tasks: [evidence span withheld — anchor status: not_found]
- [abstract] advanced network analysis scope: "analyze the ecological networks between DOM and microbes, and explore their response to warming"
