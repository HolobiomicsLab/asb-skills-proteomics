---
name: organic-matter-thermodynamic-metrics
description: Use when you have assigned molecular formulas with element counts (C,
  H, O, N, P, S) from FT-ICR MS DOM data and need thermodynamic or structural trait
  columns such as GFE, NOSC, Ymet, AImod, DBE, or kdefectCH2.
when_to_use_negative:
- Input is raw FT-ICR MS peak lists that have not yet been assigned molecular formulas
  — molecular traits require element composition per formula
- The goal is advanced ecological analyses (DOM assembly via commProc, dark matter
  via iDME, DOM-microbe networks via H2, or thermal response via iCER) rather than
  per-molecule trait calculation
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3339
tools:
- name: iDOM
  role: R package whose molTrait function computes the molecular chemical characteristics
    (mass, element ratios, AImod, DBE variants, GFE, NOSC, Ymet, kdefectCH2) from
    the molecular composition data
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed (via devtools from GitHub or
    from zip/tar.gz source) and molTrait is executed
- name: devtools
  role: Used to install the iDOM development version from GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:67440e84a3e19b9b11182d68f5a5dd267d27ab39bc791896b76b57cbab921791
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# organic-matter-thermodynamic-metrics

## Summary

Calculate thermodynamic and structural molecular traits (GFE, NOSC, carbon use efficiency Ymet, AImod, DBE variants, Kendrick defect kdefectCH2, elemental ratios) from DOM molecular formulas using the iDOM R package's molTrait function.

## When to use

Use when you have assigned molecular formulas with element counts (C, H, O, N, P, S) from FT-ICR MS DOM data and need thermodynamic or structural trait columns such as GFE, NOSC, Ymet, AImod, DBE, or kdefectCH2.

## When NOT to use

- Input is raw FT-ICR MS peak lists that have not yet been assigned molecular formulas — molecular traits require element composition per formula
- The goal is advanced ecological analyses (DOM assembly via commProc, dark matter via iDME, DOM-microbe networks via H2, or thermal response via iCER) rather than per-molecule trait calculation

## Inputs

- Molecular formula / element composition table with per-molecule C, H, O, N, P, S counts from FT-ICR MS DOM data
- Molecular intensity dataset (e.g. iDOM example DOM dataset, Tables 1 and S1)

## Outputs

- Data frame of 14 molecular trait columns per formula: mass, O/C, H/C, N/C, P/C, S/C, AImod, DBE, DBEO, DBEAI, GFE, NOSC, Ymet, kdefectCH2
- Exported trait table, e.g. mol_traits.csv

## How to apply

Install the iDOM package from GitHub with devtools::install_github('jianjunwang/iDOM') and load the package's example molecular intensity and element composition datasets (Tables 1 and S1 in the mLife paper). Run the molTrait function on the element composition table; it derives each formula's mass, elemental ratios (O/C, H/C, N/C, P/C, S/C), aromaticity index AImod, double bond equivalents DBE plus variants DBEO and DBEAI, Gibbs free energy GFE, nominal oxidation state of carbon NOSC, metabolic carbon use efficiency Ymet, and the CH2 homologous series Kendrick mass defect kdefectCH2. Inspect the returned data frame to confirm all 14 trait columns are populated for every molecular formula, then export the trait table (e.g. mol_traits.csv) for downstream chemical diversity or dissimilarity analyses in iDOM.

## Related tools

- **iDOM** (R package whose molTrait function computes the molecular chemical characteristics (mass, element ratios, AImod, DBE variants, GFE, NOSC, Ymet, kdefectCH2) from the molecular composition data) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed (via devtools from GitHub or from zip/tar.gz source) and molTrait is executed)
- **devtools** (Used to install the iDOM development version from GitHub)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); traits <- iDOM::molTrait(element_composition_data); write.csv(traits, 'mol_traits.csv', row.names = FALSE)
```

## Evaluation signals

- mol_traits.csv exists and parses as a valid CSV
- Output contains columns corresponding to all 14 molecular traits named in Table 1 (mass, O/C, H/C, N/C, P/C, S/C, AImod, DBE, DBEO, DBEAI, GFE, NOSC, Ymet, kdefectCH2)
- No missing trait values for formulas present in the input element composition dataset
- Trait row count matches the number of molecular formulas in the input data
- Element ratios (e.g. O/C, H/C) fall in chemically plausible ranges for DOM formulas

## Limitations

- Trait computation requires complete element composition per formula; uncharacterized molecules ('dark matter') without assigned formulas cannot receive traits
- The molTrait workflow is demonstrated on the package's example DOM datasets (Tables 1 and S1); adapting to other datasets requires formulas with C, H, O, N, P, S element counts

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [results] molTrait function calculates molecular chemical characteristics: [evidence span withheld — anchor status: not_found]
- [readme] installation from GitHub: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] case studies of basic functions: "case studies of basic functions for the calculation of molecular traits"
