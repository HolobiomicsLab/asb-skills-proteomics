---
name: fticr-ms-compositional-data-filtering
description: Use when an FT-ICR MS molecule-by-sample compositional matrix contains
  low-prevalence molecules and downstream statistics (e.g., Spearman MER vs. temperature,
  iCER thermal response) require a prevalence-filtered abundance table.
when_to_use_negative:
- The analysis goal is DOM assembly processes, dark matter effects, or DOM-microbe
  networks (use iDOM's commProc, iDME, or H2 functions instead of iCER-based filtering)
- The input is already a matrix of molecular traits, molecular classes, or dark matter
  rather than a raw molecule-by-sample compositional matrix
- The dataset lacks an environmental gradient (e.g., temperature) to correlate molecular
  relative abundances against, so MER computation is undefined
tools:
- name: iDOM
  role: R package providing the iCER function and example DOM compositional data used
    to filter molecules and quantify compositional-level environmental response
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and the 80:20 split, >=30%
    prevalence filtering, and Spearman MER computation are performed
- name: devtools
  role: Used to install iDOM from GitHub before filtering and iCER calculation
  repo: https://github.com/jianjunwang/iDOM
- name: FT-ICR MS
  role: Instrument generating the high-resolution molecular compositional data that
    the filtering step is applied to
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:19e54ef2232a2280576a0c69fb48a2c4f63afb8cc25d8ca744bd1a15a6f662cf
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fticr-ms-compositional-data-filtering

## Summary

Prevalence-based filtering of FT-ICR MS molecular compositional matrices (retaining molecules detected in >=30% of samples) prior to computing molecule-level environmental responses (MER) and the sample-level iCER indicator with the iDOM R package.

## When to use

Use when an FT-ICR MS molecule-by-sample compositional matrix contains low-prevalence molecules and downstream statistics (e.g., Spearman MER vs. temperature, iCER thermal response) require a prevalence-filtered abundance table.

## When NOT to use

- The analysis goal is DOM assembly processes, dark matter effects, or DOM-microbe networks (use iDOM's commProc, iDME, or H2 functions instead of iCER-based filtering)
- The input is already a matrix of molecular traits, molecular classes, or dark matter rather than a raw molecule-by-sample compositional matrix
- The dataset lacks an environmental gradient (e.g., temperature) to correlate molecular relative abundances against, so MER computation is undefined

## Inputs

- FT-ICR MS molecular compositional matrix (molecule x sample relative abundances / presence-absence)
- Sample-level environmental metadata (e.g., temperature) for the temperate and subtropical DOM datasets
- iDOM R package (source code and example dataset from https://github.com/jianjunwang/iDOM)

## Outputs

- Prevalence-filtered compositional matrix (molecules detected in >=30% of MER-subset samples)
- Molecule-level MER table (Spearman correlations of relative abundance vs. temperature, range -0.99 to 0.98)
- Sample-level iCER values and iCER-temperature regression results written to icer_mer_results.csv

## How to apply

Load the DOM molecular compositional matrix and sample temperature metadata from the iDOM GitHub package (github.com/jianjunwang/iDOM) in R, installed via devtools::install_github('jianjunwang/iDOM'). Randomly split samples 80:20 into a MER (training) subset and a held-out iCER (validation) subset before filtering so validation remains independent. In the MER subset, retain only molecules detected in >=30% of samples; the 30% prevalence cutoff removes sparse molecules whose rank correlations with temperature would be unstable. Compute each retained molecule's MER as the Spearman rank correlation between its relative abundance and temperature, then aggregate molecule-level MERs into sample-level iCER values with iDOM::iCER over 999 randomizations. Verify computed MER values fall within the reported range of -0.99 to 0.98 before writing results.

## Related tools

- **iDOM** (R package providing the iCER function and example DOM compositional data used to filter molecules and quantify compositional-level environmental response) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and the 80:20 split, >=30% prevalence filtering, and Spearman MER computation are performed)
- **devtools** (Used to install iDOM from GitHub before filtering and iCER calculation) — https://github.com/jianjunwang/iDOM
- **FT-ICR MS** (Instrument generating the high-resolution molecular compositional data that the filtering step is applied to)

## Examples

```
install.packages("devtools"); devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- Filtered matrix retains only molecules detected in >=30% of MER-subset samples (row prevalence >= 0.30)
- Computed molecule-level MER values fall within the reported range of -0.99 to 0.98
- Sample-level iCER regressed against temperature shows positive correlations with R2 = 0.76 (temperate) and R2 = 0.64 (subtropical), matching Figure 5
- 80:20 MER/iCER split and 999 randomizations are reproducible, yielding consistent iCER-temperature relationships across runs
- icer_mer_results.csv contains both the MER table and the iCER-temperature regression results

## Limitations

- The >=30% prevalence cutoff is dataset- and workflow-specific (from the iCER warming case study); other DOM datasets may need recalibration
- Uncharacterized molecules (dark matter) and the 80:20 random split introduce variability; the iCER aggregation relies on 999 randomizations rather than a deterministic calculation
- The filtering precedes correlation-based MER, so molecules near the prevalence threshold may yield unstable Spearman correlations with temperature

## Evidence

- [readme] iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM.: "iCER: Calculates the indicator of compositional-level environmental response to assess the thermal response of DOM."
- [methods] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [readme] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [intro] no software is available to integrate the advanced analyses mentioned above: "no software is available to integrate the advanced analyses mentioned above"
