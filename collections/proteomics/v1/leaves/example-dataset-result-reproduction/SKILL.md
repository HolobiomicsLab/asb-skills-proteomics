---
name: example-dataset-result-reproduction
description: Use when the goal is to reproduce a published iDOM molGroup classification
  result on the bundled warming FT-ICR MS example dataset and verify class/fraction
  composition against reported figures.
when_to_use_negative:
- Input is not an FT-ICR MS molecular formula/composition table (e.g., it is raw spectra
  or a microbial abundance table), since molGroup requires assigned molecular formulas
  and H/C–O/C traits
- The goal is a novel environmental response analysis (e.g., thermal response, assembly
  processes) rather than reproducing the published class/fraction composition — use
  iCER or commProc instead
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: iDOM
  role: R package providing the molGroup function for assigning molecular classes
    and labile/recalcitrant × active/inactive fractions, plus molecular trait calculation
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and the example warming dataset
    is loaded and analyzed
- name: devtools
  role: Used to install iDOM from its GitHub source with devtools::install_github('jianjunwang/iDOM')
- name: FT-ICR MS
  role: Instrument that produced the molecular composition data of the example warming
    DOM dataset being classified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:11f08616e0830d0c2059319d962421da7c208ed87f0fe3836af8bc2fe36a8536
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# example-dataset-result-reproduction

## Summary

Reproduces the published iDOM molGroup molecular class assignment (formula groups, biomolecule classes, and labile/recalcitrant × active/inactive fractions) on the bundled experimental warming FT-ICR MS example dataset (5°C–30°C), verifying that CHO/CHON dominate relative abundance and condensed aromatics (ConHC) and lignin dominate class composition.

## When to use

Use when the goal is to reproduce a published iDOM molGroup classification result on the bundled warming FT-ICR MS example dataset and verify class/fraction composition against reported figures.

## When NOT to use

- Input is not an FT-ICR MS molecular formula/composition table (e.g., it is raw spectra or a microbial abundance table), since molGroup requires assigned molecular formulas and H/C–O/C traits
- The goal is a novel environmental response analysis (e.g., thermal response, assembly processes) rather than reproducing the published class/fraction composition — use iCER or commProc instead

## Inputs

- Bundled iDOM example FT-ICR MS DOM dataset from the experimental warming microcosm (5°C–30°C gradient)
- Molecular formula table with elemental composition used to compute H/C and O/C molecular traits

## Outputs

- molGroup_classification.csv (molecule, molecular class, labile/recalcitrant × active/inactive fraction, relative abundance)
- Aggregated relative abundance tables by formula group (e.g., CHO, CHON) and biomolecule class (e.g., ConHC, lignin)

## How to apply

Install the iDOM package from GitHub in R with devtools::install_github('jianjunwang/iDOM'), then load the bundled example DOM dataset from the experimental warming microcosm (5°C–30°C gradient) characterized by FT-ICR MS. Compute the molecular traits (elemental ratios such as H/C and O/C) required for class assignment using iDOM's basic functions. Run the molGroup function to assign each molecule to a formula group and biomolecule class (e.g., condensed aromatics, lignin) and to the four labile/recalcitrant × active/inactive reactivity–activity fractions. Aggregate relative abundance by class and fraction, confirm that 4150 of 5474 molecules receive an assignment, and compare against Figure 2 and Figure S2 of the mLife paper before writing the classification table to molGroup_classification.csv.

## Related tools

- **iDOM** (R package providing the molGroup function for assigning molecular classes and labile/recalcitrant × active/inactive fractions, plus molecular trait calculation) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and the example warming dataset is loaded and analyzed)
- **devtools** (Used to install iDOM from its GitHub source with devtools::install_github('jianjunwang/iDOM'))
- **FT-ICR MS** (Instrument that produced the molecular composition data of the example warming DOM dataset being classified)

## Examples

```
devtools::install_github("jianjunwang/iDOM")
```

## Evaluation signals

- 4150 of 5474 molecules receive a molGroup class/fraction assignment
- CHO and CHON formula groups show the highest relative abundances across all temperatures (5°C–30°C)
- Condensed aromatics (ConHC) and lignin dominate the assigned class composition throughout the gradient
- Class/fraction composition matches the reported Figure 2 and Figure S2 of the mLife paper
- Output file molGroup_classification.csv contains molecule, class, fraction, and relative abundance columns

## Limitations

- Reproduction is specific to the bundled example warming dataset (5474 molecules); other datasets may yield different assignment coverage
- Installing iDOM requires devtools and GitHub access; the package can alternatively be installed from a local zip or tar.gz source file
- Only molecules with assigned formulas can be classified — uncharacterized 'dark matter' molecules are not part of the molGroup class assignment

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [methods] devtools install from GitHub: "devtools::install_github("jianjunwang/iDOM")"
- [readme] README about function scope: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
- [abstract] package and dataset availability: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
