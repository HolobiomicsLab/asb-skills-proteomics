---
name: elemental-composition-parsing
description: Use when you have a DOM molecular formula table (element counts C, H,
  O, N, P, S per molecule, e.g. from FT-ICR MS) and need per-molecule traits such
  as O/C, H/C, AImod, DBE, GFE, NOSC, or Ymet.
when_to_use_negative:
- Input is already a molecular trait table — molTrait expects raw element counts/formulas,
  not precomputed traits
- Data are unassigned peaks or 'dark matter' without element compositions — formula-level
  traits cannot be parsed; use iDME-type dark matter analyses instead
- The goal is advanced analyses (assembly processes, DOM-microbe networks) rather
  than per-formula trait calculation — use commProc, iDME, H2, or iCER
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3314
tools:
- name: iDOM
  role: R package providing the molTrait function that parses element compositions
    into 14 molecular traits
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which iDOM is installed and molTrait is executed
- name: devtools
  role: Used to install iDOM from GitHub (devtools::install_github('jianjunwang/iDOM'))
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:1ec91cfcc07812f6ba5aa3e28116323f1644d6a4b859cf2d723f3410593f95d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# elemental-composition-parsing

## Summary

Derives molecular chemical characteristics (mass, elemental ratios, Kendrick defect, AImod, DBE variants, GFE, NOSC, Ymet) from FT-ICR MS molecular formula/element-composition data using the iDOM R package's molTrait function.

## When to use

Use when you have a DOM molecular formula table (element counts C, H, O, N, P, S per molecule, e.g. from FT-ICR MS) and need per-molecule traits such as O/C, H/C, AImod, DBE, GFE, NOSC, or Ymet.

## When NOT to use

- Input is already a molecular trait table — molTrait expects raw element counts/formulas, not precomputed traits
- Data are unassigned peaks or 'dark matter' without element compositions — formula-level traits cannot be parsed; use iDME-type dark matter analyses instead
- The goal is advanced analyses (assembly processes, DOM-microbe networks) rather than per-formula trait calculation — use commProc, iDME, H2, or iCER

## Inputs

- Element composition table: one row per molecular formula with counts of C, H, O, N, P, S (from FT-ICR MS peak assignment)
- Molecular intensity dataset distributed with the iDOM package (e.g. the example DOM dataset)

## Outputs

- Data frame with one row per molecular formula and 14 trait columns: mass, O/C, H/C, N/C, P/C, S/C, kdefectCH2, AImod, DBE, DBEO, DBEAI, GFE, NOSC, Ymet
- Exported trait table (e.g. mol_traits.csv)

## How to apply

Install iDOM from GitHub with devtools::install_github('jianjunwang/iDOM') and load the package's example molecular intensity and element composition datasets (Tables 1 and S1 in the mLife paper). Pass the element composition data (one row per molecular formula with element counts) to the molTrait function, which parses each formula and computes 14 trait columns: mass, elemental ratios O/C, H/C, N/C, P/C, S/C, the CH2 homologous series mass defect (kdefectCH2), modified aromaticity index (AImod), double bond equivalents (DBE, DBEO, DBEAI), Gibbs free energy (GFE), nominal oxidation state of carbon (NOSC), and metabolic carbon use efficiency (Ymet). The rationale is that these derived traits bridge chemical characterization to ecological interpretation of DOM characterized by FT-ICR MS, and molTrait is the iDOM basic-analysis function for this calculation. Inspect the returned data frame to confirm all 14 trait columns are populated, then export (e.g. write.csv to mol_traits.csv) for downstream diversity, dissimilarity, or assembly analyses.

## Related tools

- **iDOM** (R package providing the molTrait function that parses element compositions into 14 molecular traits) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which iDOM is installed and molTrait is executed)
- **devtools** (Used to install iDOM from GitHub (devtools::install_github('jianjunwang/iDOM')))

## Examples

```
devtools::install_github("jianjunwang/iDOM"); traits <- iDOM::molTrait(elem_comp_data); write.csv(traits, "mol_traits.csv", row.names = FALSE)
```

## Evaluation signals

- mol_traits.csv exists and parses as a valid CSV
- Output contains columns corresponding to all 14 molecular traits named in Table 1 (mass, O/C, H/C, N/C, P/C, S/C, kdefectCH2, AImod, DBE, DBEO, DBEAI, GFE, NOSC, Ymet)
- Row count of the trait table equals the number of formulas in the input element composition dataset, with no missing trait values for input formulas
- Elemental ratios recomputed by hand for spot-check formulas (e.g. O/C = O count / C count) match the molTrait output
- kdefectCH2 values place CH2 homologous series members at consistent mass defects

## Limitations

- Traits can only be computed for formulas with assigned element compositions; uncharacterized molecules ('dark matter') in FT-ICR MS data are excluded
- iDOM is a development-version GitHub package; installation may alternatively require a local zip or tar.gz source (install.packages with repos = NULL, type = 'source')

## Evidence

- [methods] iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes,: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes,"
- [methods] devtools install: "# install.packages("devtools")
devtools::install_github("jianjunwang/iDOM")"
- [abstract] abstract package scope: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
- [readme] readme citation of molTrait scope: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
