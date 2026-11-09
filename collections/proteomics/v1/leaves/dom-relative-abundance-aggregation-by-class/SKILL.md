---
name: dom-relative-abundance-aggregation-by-class
description: Use when you have a DOM FT-ICR MS formula table (with elemental compositions
  enabling H/C and O/C molecular traits) and need molecular class assignment plus
  labile/recalcitrant active/inactive fraction classification with relative-abundance
  summaries.
when_to_use_negative:
- The input is raw mass spectra or peak lists without assigned molecular formulas
  — molGroup requires formula-level compositions to compute H/C and O/C traits
- The analysis goal is quantifying DOM assembly processes, dark matter effects, or
  DOM–microbe networks (use iDOM's commProc, iDME, or H2 functions instead)
- Assessing the compositional-level environmental (thermal) response of DOM, for which
  the iCER function is the intended tool
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: iDOM
  role: R package whose molGroup function assigns molecular classes and labile/recalcitrant
    × active/inactive fractions and supports relative-abundance aggregation for FT-ICR
    MS DOM data
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment for the iDOM package, including installing from zip/tar.gz
    source with install.packages(repos = NULL, type = 'source')
- name: devtools
  role: Used to install the development version of iDOM from its GitHub source before
    running molGroup
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:75d9615ba79e6870c43290c8d6df5c89689e13579424d54c34158bd83f7156f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dom-relative-abundance-aggregation-by-class

## Summary

Assigns FT-ICR MS-derived dissolved organic matter (DOM) molecular formulas to molecular classes (e.g., condensed aromatics, lignin) and labile/recalcitrant × active/inactive reactivity–activity fractions using the iDOM molGroup function, then aggregates relative abundance by class and fraction to identify dominant groups.

## When to use

Use when you have a DOM FT-ICR MS formula table (with elemental compositions enabling H/C and O/C molecular traits) and need molecular class assignment plus labile/recalcitrant active/inactive fraction classification with relative-abundance summaries.

## When NOT to use

- The input is raw mass spectra or peak lists without assigned molecular formulas — molGroup requires formula-level compositions to compute H/C and O/C traits
- The analysis goal is quantifying DOM assembly processes, dark matter effects, or DOM–microbe networks (use iDOM's commProc, iDME, or H2 functions instead)
- Assessing the compositional-level environmental (thermal) response of DOM, for which the iCER function is the intended tool

## Inputs

- FT-ICR MS DOM molecular formula/composition table (e.g., the iDOM example warming microcosm dataset, 5°C–30°C gradient; 5474 molecules)
- Elemental compositions sufficient to compute molecular traits (H/C and O/C ratios)

## Outputs

- Classification table mapping each molecule to its formula group, molecular class (e.g., ConHC, lignin), and labile/recalcitrant active/inactive fraction
- Relative abundance aggregated by molecular class and fraction (e.g., molGroup_classification.csv)
- Summary of dominant formula groups and classes (CHO/CHON; ConHC/lignin)

## How to apply

Install iDOM from GitHub in R with devtools::install_github('jianjunwang/iDOM'). Load the DOM formula dataset (e.g., the bundled experimental warming microcosm FT-ICR MS dataset spanning a 5°C–30°C gradient) and compute the molecular traits (elemental ratios such as H/C and O/C) that molGroup requires for class assignment. Run molGroup to assign each molecule to a molecular class (formula groups such as CHO and CHON; classes such as condensed aromatics (ConHC) and lignin) and to one of four labile/recalcitrant × active/inactive fractions. Aggregate relative abundance by molecular class and fraction, expecting only a subset of formulas to receive an assignment (in the warming example, 4150 of 5474 molecules were classified). Write the resulting molecule × class × fraction × relative-abundance table (e.g., molGroup_classification.csv) and inspect which groups dominate the composition.

## Related tools

- **iDOM** (R package whose molGroup function assigns molecular classes and labile/recalcitrant × active/inactive fractions and supports relative-abundance aggregation for FT-ICR MS DOM data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment for the iDOM package, including installing from zip/tar.gz source with install.packages(repos = NULL, type = 'source'))
- **devtools** (Used to install the development version of iDOM from its GitHub source before running molGroup)

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- The number of assigned molecules matches the expected coverage — in the warming example, 4150 of 5474 molecules receive a class/fraction assignment
- CHO and CHON formula groups consistently show the highest relative abundances across all temperature treatments (5°C–30°C)
- Condensed aromatics (ConHC) and lignin dominate the class composition throughout, reproducing the pattern in Figure 2 and Figure S2
- Every molecule in the output table carries a class label, a fraction label (labile/recalcitrant × active/inactive), and a relative abundance value
- Relative abundances within each sample/fraction aggregation sum to a coherent composition (fractions of total assigned molecules)

## Limitations

- Not all molecules receive an assignment — in the warming example, 1324 of 5474 formulas remained unclassified (dark matter), so abundance summaries cover only the assigned subset
- Classification depends on the elemental-ratio (H/C, O/C) trait thresholds used by molGroup, so results are sensitive to formula-assignment quality upstream of the FT-ICR MS table
- The validation composition reported here is specific to the iDOM example warming microcosm dataset; other DOM datasets may show different dominant classes

## Evidence

- [methods] molGroup classification workflow: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity"
- [readme] README function scope: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [readme] installation command: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] warming case study dataset: "We further showed the case studies with advanced functions to quantify DOM assembly processes, assess the effects of dark matter on molecular interactions, analyze the ecological networks between DOM"
