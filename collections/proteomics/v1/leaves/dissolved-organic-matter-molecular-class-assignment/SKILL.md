---
name: dissolved-organic-matter-molecular-class-assignment
description: Use when you have a DOM FT-ICR MS molecular formula table (elemental
  compositions, e.g., CHO/CHON/CHOS) and need each molecule assigned to a molecular
  class and to labile/recalcitrant active/inactive reactivity–activity fractions.
when_to_use_negative:
- Input is raw FT-ICR MS spectra rather than a curated molecular formula table — molecular
  traits (H/C, O/C) must first be computed from assigned formulae before molGroup
  can classify.
- The goal is advanced analyses (DOM assembly processes, dark matter effects, DOM–microbe
  networks) rather than basic molecular class assignment — those use other iDOM functions
  (commProc, iDME, H2, iCER).
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: iDOM
  role: R package whose molGroup function performs the molecular class and reactivity–activity
    fraction assignment on FT-ICR MS DOM data
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and molGroup is executed
- name: devtools
  role: Used to install iDOM from its GitHub source
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:13aa3b8e400f75f0c19f032990815a95cfe2430278d55e029e4356d526e08d8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# dissolved-organic-matter-molecular-class-assignment

## Summary

Assigns FT-ICR MS-derived dissolved organic matter (DOM) molecular formulae to formula groups (e.g., CHO, CHON), biomolecule classes (e.g., condensed hydrocarbons, lignin), and labile/recalcitrant × active/inactive reactivity–activity fractions using the molGroup function of the iDOM R package.

## When to use

Use when you have a DOM FT-ICR MS molecular formula table (elemental compositions, e.g., CHO/CHON/CHOS) and need each molecule assigned to a molecular class and to labile/recalcitrant active/inactive reactivity–activity fractions.

## When NOT to use

- Input is raw FT-ICR MS spectra rather than a curated molecular formula table — molecular traits (H/C, O/C) must first be computed from assigned formulae before molGroup can classify.
- The goal is advanced analyses (DOM assembly processes, dark matter effects, DOM–microbe networks) rather than basic molecular class assignment — those use other iDOM functions (commProc, iDME, H2, iCER).

## Inputs

- DOM FT-ICR MS molecular formula table (elemental compositions per molecule, e.g., from the iDOM bundled warming 5–30°C example dataset)
- Molecular traits computed from formulae, including H/C and O/C elemental ratios

## Outputs

- Per-molecule classification table (molecule, formula group, biomolecule class, labile/recalcitrant × active/inactive fraction, relative abundance), e.g., written to molGroup_classification.csv
- Aggregated relative-abundance summary by molecular class and by reactivity–activity fraction (e.g., 4150 of 5474 molecules assigned)

## How to apply

Install the iDOM package in R from GitHub with devtools::install_github('jianjunwang/iDOM'). Load the bundled example FT-ICR MS dataset (the experimental warming microcosm gradient, 5°C–30°C) and compute the molecular traits required for class assignment, such as elemental ratios H/C and O/C. Run molGroup to assign each formula to a molecular class (formula groups like CHO and CHON, and biomolecule classes such as condensed aromatics and lignin) and to the four labile/recalcitrant × active/inactive fractions. Aggregate relative abundances by class and by fraction, then compare the resulting composition against the reference output (Figure 2 and Figure S2). The rationale is that iDOM bridges chemical characterization and ecological interpretation by making molecular-class assignment a reproducible step in the DOM statistical pipeline.

## Related tools

- **iDOM** (R package whose molGroup function performs the molecular class and reactivity–activity fraction assignment on FT-ICR MS DOM data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and molGroup is executed)
- **devtools** (Used to install iDOM from its GitHub source)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); molGroup(ex_warming_dataset)
```

## Evaluation signals

- CHO and CHON formula groups show the highest relative abundances across the 5°C–30°C gradient.
- Condensed hydrocarbons (ConHC) and lignin dominate the assigned class composition, matching Figure 2 and Figure S2.
- 4150 of 5474 molecules receive a labile/recalcitrant × active/inactive fraction assignment.
- The output table contains one class and one fraction label per molecule plus relative abundance, matching the molGroup_classification.csv schema.
- Relative abundances aggregated by class and fraction sum consistently across the temperature-gradient samples.

## Limitations

- Only molecules whose formulae pass trait computation receive a class/fraction assignment; in the reference run, 5474 − 4150 = 1324 molecules were left unclassified.
- Class and fraction definitions are those implemented in iDOM/molGroup; results are reproducible only against the bundled warming example dataset and published figures, not against arbitrary external benchmarks.
- iDOM's source is installed from GitHub without an explicit license assertion, so redistribution terms are unclear.

## Evidence

- [methods] molGroup example run: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [readme] installation: "devtools::install_github("jianjunwang/iDOM")"
- [readme] package scope: "the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and dissimilarity"
- [abstract] reproducibility source: "The source code and example dataset of iDOM are publicly available on https://github.com/jianjunwang/iDOM"
- [intro] classification basis: "no software is available to integrate the advanced analyses mentioned above"
