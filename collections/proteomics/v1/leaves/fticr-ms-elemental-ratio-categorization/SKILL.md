---
name: fticr-ms-elemental-ratio-categorization
description: Use when an FT-ICR MS DOM peak/formula table (e.g., molecular formulae
  with elemental composition) needs categorical molecular class and reactivity–activity
  fraction assignment for compositional summarization.
when_to_use_negative:
- The data are raw FT-ICR MS spectra or unassigned peaks without molecular formulae
  — formula assignment must precede class categorization
- The goal is quantifying DOM assembly processes, dark-matter effects, or DOM–microbe
  networks — iDOM's commProc, iDME, or H2 functions, not molGroup, are the appropriate
  entry points
- The input is already a categorized class/fraction table; recomputation would not
  add information
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: iDOM
  role: Provides the molGroup function and bundled example warming FT-ICR MS dataset
    used to compute molecular traits and assign molecular classes and reactivity–activity
    fractions
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed from source and molGroup is
    executed
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install the iDOM development version from GitHub (devtools::install_github)
  repo: https://github.com/jianjunwang/iDOM
- name: FT-ICR MS
  role: Instrument that generates the molecular compositional data (formulae characterized
    by high-resolution mass spectrometry) that serve as input to class assignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:f5f331a809c3ec3587c829ee00f7074c292618ca2d161b01c977e5ddf0bb0746
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# fticr-ms-elemental-ratio-categorization

## Summary

Assigns FT-ICR MS-derived molecular formulae to DOM molecular classes (e.g., condensed aromatics/ConHC, lignin, CHO/CHON formula groups) and labile/recalcitrant × active/inactive reactivity–activity fractions using iDOM's molGroup function in R, based on molecular traits such as H/C and O/C ratios.

## When to use

Use when an FT-ICR MS DOM peak/formula table (e.g., molecular formulae with elemental composition) needs categorical molecular class and reactivity–activity fraction assignment for compositional summarization.

## When NOT to use

- The data are raw FT-ICR MS spectra or unassigned peaks without molecular formulae — formula assignment must precede class categorization
- The goal is quantifying DOM assembly processes, dark-matter effects, or DOM–microbe networks — iDOM's commProc, iDME, or H2 functions, not molGroup, are the appropriate entry points
- The input is already a categorized class/fraction table; recomputation would not add information

## Inputs

- FT-ICR MS DOM molecular formula table (molecule/compositional data with elemental formulae, e.g., the bundled experimental warming 5–30°C dataset)
- Molecular traits derived from formulae (elemental ratios such as H/C and O/C)

## Outputs

- Per-molecule classification table (molecule, molecular class, labile/recalcitrant × active/inactive fraction, relative abundance), e.g., molGroup_classification.csv
- Aggregated relative-abundance summaries by molecular class (e.g., ConHC, lignin, CHO, CHON groups) and by reactivity–activity fraction

## How to apply

Install the iDOM R package from GitHub with devtools::install_github('jianjunwang/iDOM') (or from a local zip/tar.gz via install.packages with repos = NULL, type = 'source'). Load the bundled example warming FT-ICR MS dataset (5–30°C gradient) and compute the molecular traits — elemental ratios such as H/C and O/C — that molGroup requires for class assignment. Run molGroup to assign each molecule to a molecular class and to the four labile/recalcitrant × active/inactive fractions. Aggregate relative abundance by class and fraction, checking that 4150 of 5474 molecules receive an assignment, CHO and CHON dominate relative abundance, and condensed aromatics (ConHC) and lignin dominate class composition, as in Figure 2 and Figure S2. Write the classification table (molecule, class, fraction, relative abundance) to a CSV such as molGroup_classification.csv.

## Related tools

- **iDOM** (Provides the molGroup function and bundled example warming FT-ICR MS dataset used to compute molecular traits and assign molecular classes and reactivity–activity fractions) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed from source and molGroup is executed) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install the iDOM development version from GitHub (devtools::install_github)) — https://github.com/jianjunwang/iDOM
- **FT-ICR MS** (Instrument that generates the molecular compositional data (formulae characterized by high-resolution mass spectrometry) that serve as input to class assignment)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); classes <- iDOM::molGroup(example_warming_data)
```

## Evaluation signals

- 4150 of 5474 molecules in the example warming dataset receive a class/fraction assignment
- CHO and CHON formula groups show the highest relative abundances across the 5–30°C temperature gradient
- Condensed aromatics (ConHC) and lignin dominate class composition, matching Figure 2 and Figure S2
- Every assigned molecule carries a class label and one of the four labile/recalcitrant × active/inactive fractions
- The written classification table contains molecule, class, fraction, and relative abundance columns

## Limitations

- Class assignment depends on prior calculation of molecular traits (elemental ratios such as H/C and O/C); molecules lacking assignable traits remain unclassified (only 4150 of 5474 example molecules were assigned)
- iDOM must be installed from source (GitHub or a local zip/tar.gz with repos = NULL, type = 'source'), which can complicate installation in restricted environments
- Validation of composition depends on the bundled example warming dataset; other DOM datasets may show different dominant classes

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [abstract] The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter).: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [readme] devtools::install_github("jianjunwang/iDOM"): "devtools::install_github("jianjunwang/iDOM")"
- [readme] install.packages("file_path/package_file_name.extension",repos = NULL, type = "source"): "install.packages("file_path/package_file_name.extension",repos = NULL, type = "source")"
