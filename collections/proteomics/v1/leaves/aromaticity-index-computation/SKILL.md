---
name: aromaticity-index-computation
description: Use when you have FT-ICR MS molecular formula lists (element composition
  with C/H/O/N/P/S) and need aromaticity classification or DBE-based traits for dissolved
  organic matter molecules.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or peaks without assigned molecular formulas — traits
  cannot be computed on uncharacterized 'dark matter' molecules
- The goal is molecular class assignment, chemical diversity/dissimilarity, or DOM-microbe
  network analyses (commProc, iDME, H2, iCER) — these are separate iDOM functions,
  not molTrait
- The dataset is not molecular compositional DOM data (e.g. taxonomic abundance tables)
edam_operation: http://edamontology.org/operation_0241
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: iDOM
  role: Provides the molTrait function that computes AImod, DBE, DBEO, and DBEAI from
    element composition data of FT-ICR MS-characterized DOM
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which the iDOM package and molTrait are executed
- name: devtools
  role: Used to install iDOM from GitHub (devtools::install_github) before running
    molTrait
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:a46655cf331b2c4c8a8a83f42c544cd221644d16d921286f578c8044ef75fcd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0241
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# aromaticity-index-computation

## Summary

Computes the modified aromaticity index (AImod) and double-bond-equivalent traits (DBE, DBEO, DBEAI) from DOM molecular formulas as part of iDOM's molTrait function, turning FT-ICR MS molecular compositional data into chemical characteristic descriptors.

## When to use

Use when you have FT-ICR MS molecular formula lists (element composition with C/H/O/N/P/S) and need aromaticity classification or DBE-based traits for dissolved organic matter molecules.

## When NOT to use

- Input is raw FT-ICR MS spectra or peaks without assigned molecular formulas — traits cannot be computed on uncharacterized 'dark matter' molecules
- The goal is molecular class assignment, chemical diversity/dissimilarity, or DOM-microbe network analyses (commProc, iDME, H2, iCER) — these are separate iDOM functions, not molTrait
- The dataset is not molecular compositional DOM data (e.g. taxonomic abundance tables)

## Inputs

- Element composition table of assigned DOM molecular formulas (C, H, O, N, P, S counts), as in the package's Tables 1 and S1
- Molecular intensity dataset from FT-ICR MS characterization distributed with the iDOM package

## Outputs

- Data frame of 14 molecular trait columns per formula, including AImod, DBE, DBEO, DBEAI, mass, elemental ratios, kdefectCH2, GFE, NOSC, and Ymet
- CSV export of the trait table (e.g. mol_traits.csv)

## How to apply

Install the iDOM R package from GitHub with devtools::install_github('jianjunwang/iDOM') (or install.packages from a local zip/tar.gz with repos = NULL, type = 'source'). Load the molecular intensity and element composition example datasets distributed with the package. Call molTrait on the element composition table: it derives mass, elemental ratios (O/C, H/C, N/C, P/C, S/C), the CH2 homologous-series mass defect (kdefectCH2), aromaticity index AImod, DBE and its variants DBEO and DBEAI, Gibbs free energy (GFE), nominal oxidation state of carbon (NOSC), and metabolic carbon-use efficiency (Ymet) — 14 trait columns in total, per Table 1. Inspect the returned data frame to confirm the AImod/DBE columns are populated for every input formula, then export (e.g. write.csv to mol_traits.csv).

## Related tools

- **iDOM** (Provides the molTrait function that computes AImod, DBE, DBEO, and DBEAI from element composition data of FT-ICR MS-characterized DOM) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which the iDOM package and molTrait are executed)
- **devtools** (Used to install iDOM from GitHub (devtools::install_github) before running molTrait)

## Examples

```
library(iDOM); traits <- molTrait(element_data); write.csv(traits, 'mol_traits.csv')
```

## Evaluation signals

- mol_traits.csv exists, parses as valid CSV, and contains a column corresponding to each of the 14 traits named in Table 1, including AImod, DBE, DBEO, and DBEAI
- AImod and DBE-variant columns have no missing values for any molecular formula present in the input element composition dataset
- AImod values are bounded (0–1 by construction of the modified aromaticity index); negative or NA values indicate malformed formulas
- Re-running molTrait on the same package datasets reproduces identical trait values
- Every formula in the output matches a formula in the input element composition table (row count preserved)

## Limitations

- Traits can only be computed for molecules with fully assigned elemental formulas; uncharacterized molecules (dark matter) yield no aromaticity or DBE values
- Package is installed from GitHub (devtools) or local zip/tar.gz source, so version drift or build failures in the development version may affect reproducibility
- The trait set is fixed by the package's Tables 1 and S1; alternative aromaticity metrics beyond AImod/DBE variants are not provided

## Evidence

- [readme] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits"
- [methods] devtools install command: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] package scope statement: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
- [readme] pipeline framing: "iDOM will serve as a comprehensive pipeline for DOM statistical analyses and bridge the gap between chemical characterization and ecological interpretation."
