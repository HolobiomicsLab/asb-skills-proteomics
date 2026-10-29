---
name: sample-subset-random-splitting
description: 'Use when computing the iDOM iCER indicator of compositional-level environmental
  response: the Spearman MERs must be trained on one random sample subset and validated
  on an independent held-out subset.'
when_to_use_negative:
- When the analysis target is a DOM assembly-process (commProc), dark-matter (iDME),
  or DOM-microbe network (H2) quantity that uses the full sample set and does not
  require a held-out validation subset
- When the sample count is too small for an 80:20 split to leave a meaningful validation
  subset for correlating iCER with the environmental gradient
- When molecules are filtered differently than the >=30% detection threshold used
  for MER training, since MERs and iCER must be computed on consistently defined molecule
  sets
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: iDOM
  role: R package providing the iCER function that aggregates molecule-level MERs
    into the sample-level indicator on the held-out validation subset
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Environment in which the sample split, MER computation, and iCER aggregation
    are performed
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install iDOM from GitHub before running the split-and-validate workflow
  repo: github.com/jianjunwang/iDOM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:b570a2c0f0c6ccf38c5674dff1c69407464045f826345d80554f41dc66f6262e
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sample-subset-random-splitting

## Summary

Randomly splits a DOM molecular compositional sample set into a training subset (for computing molecule-level MERs) and a held-out validation subset (for computing sample-level iCER values), preventing overfitting of the compositional-level environmental response indicator. In the iDOM thermal-response workflow the split is 80:20 with 999 randomizations.

## When to use

Use when computing the iDOM iCER indicator of compositional-level environmental response: the Spearman MERs must be trained on one random sample subset and validated on an independent held-out subset.

## When NOT to use

- When the analysis target is a DOM assembly-process (commProc), dark-matter (iDME), or DOM-microbe network (H2) quantity that uses the full sample set and does not require a held-out validation subset
- When the sample count is too small for an 80:20 split to leave a meaningful validation subset for correlating iCER with the environmental gradient
- When molecules are filtered differently than the >=30% detection threshold used for MER training, since MERs and iCER must be computed on consistently defined molecule sets

## Inputs

- DOM molecular compositional matrix (samples x molecules, relative abundances) from FT-ICR MS
- Sample temperature metadata (e.g. 5-30°C experimental warming gradient)
- Randomization seed/count (999 randomizations) and split proportion (80:20)

## Outputs

- MER (training) subset and iCER (validation) subset sample assignments
- Molecule-level MER table (Spearman correlations, reported range -0.99 to 0.98)
- Sample-level iCER values for the held-out subset aggregated over 999 randomizations
- iCER-temperature regression results (R2 = 0.76 temperate, 0.64 subtropical) written e.g. to icer_mer_results.csv

## How to apply

Load the DOM molecular compositional matrix and sample temperature metadata (e.g. from the iDOM GitHub example dataset in R), then randomly split the samples 80:20 into a MER (training) subset and an iCER (validation) subset. In the MER subset, retain only molecules detected in >=30% of samples and compute each molecule's MER as the Spearman rank correlation between its relative abundance and temperature. Apply the iDOM::iCER function to aggregate the molecule-level MERs onto the held-out iCER subset, repeating over 999 randomizations so the indicator does not depend on a single arbitrary split. Finally, regress sample-level iCER against temperature per region to confirm the split did not bias the environmental-response signal.

## Related tools

- **iDOM** (R package providing the iCER function that aggregates molecule-level MERs into the sample-level indicator on the held-out validation subset) — https://github.com/jianjunwang/iDOM
- **R** (Environment in which the sample split, MER computation, and iCER aggregation are performed) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install iDOM from GitHub before running the split-and-validate workflow) — github.com/jianjunwang/iDOM

## Examples

```
devtools::install_github("jianjunwang/iDOM"); icer <- iDOM::iCER(mer, icER_subset, n = 999)
```

## Evaluation signals

- Sample-level iCER values regressed against temperature show positive correlations with R2 of 0.76 (temperate) and 0.64 (subtropical), matching Figure 5
- Computed molecule-level MER values fall within the reported range of -0.99 to 0.98
- The split is reproducible across 999 randomizations, i.e. iCER-temperature relationships are consistent rather than split-dependent
- Only molecules detected in >=30% of MER-subset samples enter the MER correlation step
- MER table and iCER-temperature regression results are written to icer_mer_results.csv with correct sample-to-subset assignments

## Limitations

- The 80:20 split is arbitrary per draw; a single split can bias iCER, hence the 999-randomization aggregation
- Molecules detected in <30% of samples are excluded from MER estimation, so rare molecules contribute no signal to the validation indicator
- The validation result depends on the availability of an environmental gradient (temperature) and was demonstrated only for temperate and subtropical warming datasets

## Evidence

- [other] Randomly split the samples 80:20 into a MER (training) subset and an iCER (validation) subset.: [evidence span withheld — anchor status: not_found]
- [methods] iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [methods] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [results] iCER values exhibited consistent warming responses in the temperate and subtropical regions, with positive correlations with temperature in both regions.: [evidence span withheld — anchor status: not_found]
- [readme] The source code and example dataset of iDOM are publicly available on https://github.com/jianjunwang/iDOM: "The source code and example dataset of iDOM are publicly available on https://github.com/jianjunwang/iDOM"
