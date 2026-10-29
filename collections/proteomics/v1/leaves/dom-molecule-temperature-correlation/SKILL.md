---
name: dom-molecule-temperature-correlation
description: Use when FT-ICR MS DOM molecular compositional matrices are paired with
  per-sample temperature (or another environmental gradient) metadata and the goal
  is to quantify thermal responses at both the molecular (MER) and compositional (iCER)
  levels.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or peak lists rather than a processed molecular compositional
  matrix aligned to sample metadata — molecular traits/class assignment must be done
  first with iDOM basic functions
- The environmental variable is not a continuous gradient measurable per sample (e.g.,
  categorical treatment labels only), since MERs are Spearman rank correlations against
  temperature
- The goal is DOM assembly processes (commProc), dark matter effects (iDME), or DOM-microbe
  network specialization (H2) — those iDOM functions address different questions
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3272
tools:
- name: iDOM
  role: R package providing the iCER function that aggregates molecule-level MERs
    into the sample-level indicator of compositional-level environmental response
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed (via devtools::install_github)
    and MER Spearman correlations plus iCER-temperature regressions are computed
- name: devtools
  role: Used to install the iDOM development version from GitHub before running the
    MER/iCER workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:9b299bc47830546ef03bf15d0b87adee345daaf0011e144461764e5c320c443f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dom-molecule-temperature-correlation

## Summary

Computes molecule-level environmental responses (MERs) as Spearman rank correlations between FT-ICR MS-derived molecular relative abundances and temperature, then aggregates them with the iDOM::iCER function into a sample-level indicator of compositional-level environmental response of DOM assemblages.

## When to use

Use when FT-ICR MS DOM molecular compositional matrices are paired with per-sample temperature (or another environmental gradient) metadata and the goal is to quantify thermal responses at both the molecular (MER) and compositional (iCER) levels.

## When NOT to use

- Input is raw FT-ICR MS spectra or peak lists rather than a processed molecular compositional matrix aligned to sample metadata — molecular traits/class assignment must be done first with iDOM basic functions
- The environmental variable is not a continuous gradient measurable per sample (e.g., categorical treatment labels only), since MERs are Spearman rank correlations against temperature
- The goal is DOM assembly processes (commProc), dark matter effects (iDME), or DOM-microbe network specialization (H2) — those iDOM functions address different questions

## Inputs

- DOM molecular compositional matrix (molecules x samples of FT-ICR MS relative abundances/presence) from the iDOM example dataset
- Sample-level temperature metadata table matching the compositional matrix columns
- Region/ grouping metadata (e.g., temperate vs. subtropical) for stratified regression

## Outputs

- MER table of per-molecule Spearman rank correlations between relative abundance and temperature (reported range -0.99 to 0.98), written to icer_mer_results.csv
- Sample-level iCER values computed for the held-out validation subset over 999 randomizations
- iCER-vs-temperature regression results per region (e.g., R2 = 0.76 temperate, 0.64 subtropical)

## How to apply

Load the DOM molecular compositional matrix and sample temperature metadata from the iDOM GitHub package and randomly split samples 80:20 into a MER (training) subset and an iCER (validation) subset. In the MER subset, retain only molecules detected in >=30% of samples and compute each molecule's MER as the Spearman rank correlation between its relative abundance and temperature. Pass these molecule-level MERs, together with the held-out iCER subset, to the iDOM::iCER function to aggregate them into a sample-level indicator of compositional-level environmental response over 999 randomizations. Finally, regress sample-level iCER against temperature separately per region (e.g., temperate and subtropical) to test whether the compositional thermal response is positively correlated with warming.

## Related tools

- **iDOM** (R package providing the iCER function that aggregates molecule-level MERs into the sample-level indicator of compositional-level environmental response) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed (via devtools::install_github) and MER Spearman correlations plus iCER-temperature regressions are computed)
- **devtools** (Used to install the iDOM development version from GitHub before running the MER/iCER workflow)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); library(iDOM); icer <- iCER(mer = mer_table, comm = dom_validation_matrix, n = 999)
```

## Evaluation signals

- MER values fall within the reported range of -0.99 to 0.98
- Sample-level iCER shows a positive correlation with temperature, with R2 near 0.76 for the temperate dataset and 0.64 for the subtropical dataset (Figure 5)
- Molecules retained for MER computation were detected in >=30% of MER-subset samples
- iCER computed on the held-out 20% validation subset over 999 randomizations, confirming generalization beyond the training (MER) subset
- icer_mer_results.csv contains both the MER table and the iCER-temperature regression results

## Limitations

- Requires a valid temperature (or other continuous environmental) measurement per sample; MERs are rank correlations against that gradient
- Molecules detected in <30% of samples are excluded, so sparse molecular features are dropped from the response indicator
- The 80:20 MER/iCER split is random, so results depend on the split; 999 randomizations are used to stabilize the iCER estimate
- Reported positive correlations were demonstrated for temperate and subtropical regions; other regions may behave differently

## Evidence

- [methods] iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [methods] devtools::install_github: "devtools::install_github("jianjunwang/iDOM")"
- [readme] iDOM README iCER description: "[iCER](https://www.nature.com/articles/s41467-024-44813-2): Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [results] iCER values exhibited consistent warming responses in the temperate and subtropical regions, with positive correlations with temperature in both regions.: [evidence span withheld — anchor status: not_found]
