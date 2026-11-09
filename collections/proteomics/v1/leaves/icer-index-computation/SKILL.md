---
name: icer-index-computation
description: Use when you have a DOM molecular compositional matrix (FT-ICR MS) plus
  per-sample environmental metadata (e.g., temperature) and need a sample-level indicator
  of compositional-level environmental response, such as DOM thermal response to experimental
  warming.
when_to_use_negative:
- Input lacks paired environmental/explanatory metadata (e.g., temperature) for samples
  — iCER quantifies response to an environmental gradient and cannot be computed without
  it
- The goal is DOM assembly process quantification, dark matter effects, or DOM-microbe
  network specialization — use iDOM's commProc, iDME, or H2 functions instead
- Data are raw FT-ICR MS spectra not yet reduced to a molecular compositional matrix
  — molecular formula assignment must precede iCER computation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the iCER function that calculates the indicator of compositional-level
    environmental response to assess the thermal response of DOM
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed (devtools::install_github)
    and MER/iCER computations and regressions are executed
- name: devtools
  role: Used to install the iDOM development version from GitHub before calling iCER
- name: FT-ICR MS
  role: Instrument whose high-resolution mass spectrometry output provides the molecular
    compositional data that iCER is computed from
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:e392f99ea1da6da154240027c4e0252209a5259bfe454ae9fc0afcdd92f2d6c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# icer-index-computation

## Summary

Computation of the iCER (indicator of compositional-level environmental response) from the iDOM R package, which aggregates molecule-level environmental response correlations into a sample-level index to assess the thermal response of dissolved organic matter (DOM) assemblages characterized by FT-ICR MS.

## When to use

Use when you have a DOM molecular compositional matrix (FT-ICR MS) plus per-sample environmental metadata (e.g., temperature) and need a sample-level indicator of compositional-level environmental response, such as DOM thermal response to experimental warming.

## When NOT to use

- Input lacks paired environmental/explanatory metadata (e.g., temperature) for samples — iCER quantifies response to an environmental gradient and cannot be computed without it
- The goal is DOM assembly process quantification, dark matter effects, or DOM-microbe network specialization — use iDOM's commProc, iDME, or H2 functions instead
- Data are raw FT-ICR MS spectra not yet reduced to a molecular compositional matrix — molecular formula assignment must precede iCER computation

## Inputs

- DOM molecular compositional matrix (molecules x samples relative abundances/presence from FT-ICR MS)
- Sample-level environmental metadata table (e.g., temperature per sample)
- Region/group labels (e.g., temperate vs. subtropical) for stratified regression

## Outputs

- Molecule-level MER table (Spearman rank correlations of relative abundance vs. temperature, expected range approx. -0.99 to 0.98)
- Sample-level iCER values for the validation subset
- iCER-vs-temperature regression results per region (e.g., R2 of 0.76 temperate and 0.64 subtropical)
- Results table (e.g., icer_mer_results.csv) with MER values and regression outputs

## How to apply

Install iDOM from GitHub (devtools::install_github("jianjunwang/iDOM")) in R. Load the DOM molecular compositional matrix and sample temperature metadata, then randomly split samples 80:20 into a MER (training) subset and an iCER (validation) subset. In the MER subset, retain only molecules detected in >=30% of samples and compute each molecule's MER as the Spearman rank correlation between its relative abundance and temperature. Call iDOM::iCER to aggregate the molecule-level MERs into a sample-level indicator for the held-out iCER subset (e.g., over 999 randomizations), then regress sample-level iCER against temperature per region (e.g., temperate and subtropical separately) to test the thermal response. The held-out split guards against overfitting the molecule-environment correlations before aggregation.

## Related tools

- **iDOM** (R package providing the iCER function that calculates the indicator of compositional-level environmental response to assess the thermal response of DOM) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed (devtools::install_github) and MER/iCER computations and regressions are executed)
- **devtools** (Used to install the iDOM development version from GitHub before calling iCER)
- **FT-ICR MS** (Instrument whose high-resolution mass spectrometry output provides the molecular compositional data that iCER is computed from)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); library(iDOM); icer <- iDOM::iCER(mer = mer_table, comm = dom_validation_matrix, n = 999)
```

## Evaluation signals

- Molecule-level MER (Spearman correlation) values fall within the reported range of -0.99 to 0.98
- Sample-level iCER regressed against temperature shows positive correlations within each region, matching reported R2 of 0.76 (temperate) and 0.64 (subtropical)
- iCER is computed on the held-out 20% validation subset, not the MER training subset used for correlation fitting
- Molecules retained for MER computation are detected in >=30% of samples in the training subset
- MER table and iCER-temperature regression results are written to a complete results file (e.g., icer_mer_results.csv) without missing iCER values

## Limitations

- iCER was demonstrated on experimental warming datasets across a 5-30°C gradient in temperate and subtropical regions; transfer to other gradients or biomes is untested in this context
- Requires a random training/validation (80:20) split; results depend on randomization (e.g., 999 randomizations), so repeated runs may vary without a fixed seed
- Molecules detected in <30% of samples are excluded, so results are sensitive to detection-frequency filtering choices

## Evidence

- [methods] iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [methods] devtools::install_github: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM"
- [readme] The source code and example dataset of iDOM are publicly available on https://github.com/jianjunwang/iDOM: "The source code and example dataset of iDOM are publicly available on https://github.com/jianjunwang/iDOM"
