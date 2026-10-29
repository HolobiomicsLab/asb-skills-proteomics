---
name: molecular-trait-calculation
description: Use when starting from FT-ICR MS-derived molecular formula assignments
  (element compositions with intensities) and you need chemically interpretable per-molecule
  traits such as O/C, H/C, N/C, P/C, S/C ratios, AImod, DBE, GFE, NOSC, or Ymet.
when_to_use_negative:
- The input is raw mass-spectra (e.g. centroided peaks) rather than assigned molecular
  formulas — formula/element composition assignment must happen first.
- The goal is one of iDOM's advanced analyses (commProc assembly processes, iDME dark
  matter, H2 network specialization, iCER environmental response) rather than per-molecule
  trait computation — those functions have separate workflows.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: iDOM
  role: R package whose molTrait function computes the 14 molecular traits from the
    element composition dataset
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Execution environment in which the iDOM package is installed and molTrait
    is run
- name: devtools
  role: Used to install iDOM from its GitHub source repository
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:849ba70313387407cfd98b17497d3c8d1eefea23e91277d6100a5aa773477ea4
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
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# molecular-trait-calculation

## Summary

Computes per-molecule chemical trait values (mass, element ratios, Kendrick defect, AImod, DBE variants, GFE, NOSC, carbon use efficiency Ymet) from DOM molecular formula data using the iDOM R package's molTrait function, as described in Meng et al. 2025 (mLife).

## When to use

Use when starting from FT-ICR MS-derived molecular formula assignments (element compositions with intensities) and you need chemically interpretable per-molecule traits such as O/C, H/C, N/C, P/C, S/C ratios, AImod, DBE, GFE, NOSC, or Ymet.

## When NOT to use

- The input is raw mass-spectra (e.g. centroided peaks) rather than assigned molecular formulas — formula/element composition assignment must happen first.
- The goal is one of iDOM's advanced analyses (commProc assembly processes, iDME dark matter, H2 network specialization, iCER environmental response) rather than per-molecule trait computation — those functions have separate workflows.

## Inputs

- Molecular formula / element composition table (per-formula counts of C, H, O, N, S, P) from FT-ICR MS, e.g. iDOM Table S1 example dataset
- Molecular intensity (relative abundance) table matching the formula list, e.g. iDOM Table 1 example dataset
- R environment with devtools and the iDOM package installed from GitHub

## Outputs

- A data frame / CSV (e.g. mol_traits.csv) with one row per molecular formula and 14 trait columns: mass, O/C, H/C, N/C, P/C, S/C, kdefectCH2, AImod, DBE, DBEO, DBEAI, GFE, NOSC, Ymet

## How to apply

Install iDOM from GitHub with devtools::install_github('jianjunwang/iDOM') and load the package's example molecular intensity and element composition datasets (Tables 1 and S1 of Meng et al. 2025). Run molTrait on the element composition data frame (one row per molecular formula with counts of C, H, O, N, S, P) so it derives the 14 traits: mass, elemental ratios O/C, H/C, N/C, P/C, S/C, the CH2 homologous series Kendrick mass defect (kdefectCH2), the modified aromaticity index (AImod), double bond equivalents DBE, DBEO, DBEAI, calculated Gibbs free energy (GFE), nominal oxidation state of carbon (NOSC), and metabolic carbon use efficiency (Ymet). The rationale is that these formulas-based traits (aromaticity, saturation, oxidation state, thermodynamic energy) are the standard currency for linking FT-ICR MS molecular composition to ecological interpretation. Inspect the returned data frame to confirm all trait columns are populated per formula, then export as a table (e.g. mol_traits.csv) for downstream iDOM analyses such as chemical diversity, dissimilarity, or assembly-process quantification.

## Related tools

- **iDOM** (R package whose molTrait function computes the 14 molecular traits from the element composition dataset) — https://github.com/jianjunwang/iDOM
- **R** (Execution environment in which the iDOM package is installed and molTrait is run)
- **devtools** (Used to install iDOM from its GitHub source repository)

## Examples

```
devtools::install_github('jianjunwang/iDOM'); traits <- iDOM::molTrait(element_composition); write.csv(traits, 'mol_traits.csv', row.names = FALSE)
```

## Evaluation signals

- mol_traits.csv exists and parses as a valid CSV
- The output contains columns corresponding to all 14 traits named in Table 1 (mass, O/C, H/C, N/C, P/C, S/C, kdefectCH2, AImod, DBE, DBEO, DBEAI, GFE, NOSC, Ymet)
- Every molecular formula present in the input element composition table has non-missing values for all trait columns
- Element-ratio columns (O/C, H/C, N/C, P/C, S/C) match hand-computed ratios of element counts for a spot-checked formula
- Row count of the trait table equals the row count of the input formula table

## Limitations

- molTrait operates only on already-assigned molecular formulas; uncharacterized molecules ('dark matter') cannot receive trait values and are excluded from the trait table.
- Trait set is fixed by the iDOM implementation (the 14 Table 1 traits); traits not in that set require custom calculation outside the package.

## Evidence

- [methods] molTrait computes traits: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes"
- [abstract] package scope: "we developed an R package iDOM with functions for the basic and advanced statistical analyses and the visualization of DOM"
- [readme] installation: "devtools::install_github("jianjunwang/iDOM")"
- [abstract] data types handled: "The package could handle various data types of DOM, including molecular compositional data, molecular traits, and uncharacterized molecules (i.e., dark matter)."
