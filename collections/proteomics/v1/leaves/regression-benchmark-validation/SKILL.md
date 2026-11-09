---
name: regression-benchmark-validation
description: Use when a workflow claims to reproduce published regression results
  (e.g., iCER thermal-response correlations for DOM assemblages along a 5-30°C warming
  gradient) and requires numeric verification against reported R², value ranges, and
  randomization counts.
when_to_use_negative:
- Input is raw FT-ICR MS peak data rather than a processed molecular compositional
  matrix with sample temperature metadata
- The analysis goal is DOM assembly-process quantification (commProc) or dark-matter
  network effects (iDME) rather than compositional-level environmental response
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iDOM
  role: R package providing the iCER function that aggregates molecule-level MERs
    into a sample-level indicator of compositional-level environmental response
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and the MER/iCER regression
    validation is executed
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install the iDOM development version from GitHub before running the
    validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:366660085d102887d3a975f5a9c6ca39a72281ff287b87e46e60f21a9c0f337a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# regression-benchmark-validation

## Summary

Validates a regression-based skill by reproducing published benchmark results: train molecule-level MER (Spearman rank correlations with temperature) on an 80% training subset, compute sample-level iCER on the held-out subset, and check the resulting iCER-vs-temperature regressions against reported R-squared values.

## When to use

Use when a workflow claims to reproduce published regression results (e.g., iCER thermal-response correlations for DOM assemblages along a 5-30°C warming gradient) and requires numeric verification against reported R², value ranges, and randomization counts.

## When NOT to use

- Input is raw FT-ICR MS peak data rather than a processed molecular compositional matrix with sample temperature metadata
- The analysis goal is DOM assembly-process quantification (commProc) or dark-matter network effects (iDME) rather than compositional-level environmental response

## Inputs

- DOM molecular compositional matrix (molecule x sample relative abundance table) from the iDOM example dataset
- Sample temperature metadata (5-30°C experimental warming gradient, temperate and subtropical regions)

## Outputs

- Molecule-level MER table (Spearman correlations, expected range -0.99 to 0.98)
- Sample-level iCER values for the held-out validation subset (999 randomizations)
- iCER-vs-temperature regression results per region (expected R² 0.76 temperate, 0.64 subtropical), written to icer_mer_results.csv

## How to apply

Load the DOM molecular compositional matrix and sample temperature metadata from the iDOM GitHub package, then randomly split samples 80:20 into a MER (training) subset and an iCER (validation) subset. In the MER subset, retain only molecules detected in >=30% of samples and compute each molecule's MER as the Spearman rank correlation between its relative abundance and temperature. Call the iDOM::iCER function to aggregate molecule-level MERs into a sample-level indicator of compositional-level environmental response for the held-out subset over 999 randomizations. Regress sample-level iCER against temperature separately for temperate and subtropical datasets and confirm positive correlations with R² of 0.76 (temperate) and 0.64 (subtropical) as reported in Figure 5. Verify computed MER values fall within the reported range of -0.99 to 0.98 before writing the MER table and iCER-temperature regression results to icer_mer_results.csv.

## Related tools

- **iDOM** (R package providing the iCER function that aggregates molecule-level MERs into a sample-level indicator of compositional-level environmental response) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and the MER/iCER regression validation is executed) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the iDOM development version from GitHub before running the validation)

## Examples

```
install.packages("devtools"); devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Temperate-region iCER-vs-temperature regression yields positive correlation with R² = 0.76
- Subtropical-region iCER-vs-temperature regression yields positive correlation with R² = 0.64
- All computed molecule-level MER values fall within the reported range of -0.99 to 0.98
- iCER values are computed over exactly 999 randomizations on the 20% held-out subset
- Output files (MER table and regression results) are written to icer_mer_results.csv with correct schema

## Limitations

- Requires a processed FT-ICR MS molecular compositional matrix paired with temperature metadata; cannot run on uncharacterized dark matter alone
- The iCER indicator relies on a random 80:20 train/validation split, so results may vary across randomizations unless seeded
- Benchmark R² values (0.76 temperate, 0.64 subtropical) are specific to the warming-gradient datasets of the source study and may not generalize to other regions or gradients

## Evidence

- [methods] iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [methods] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [readme] iCER (https://www.nature.com/articles/s41467-024-44813-2): Calculates the indicator of compositional-level environmental response: "iCER (https://www.nature.com/articles/s41467-024-44813-2): Calculates the indicator of compositional-level environmental response"
