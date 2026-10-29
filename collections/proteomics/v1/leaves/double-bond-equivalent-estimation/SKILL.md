---
name: double-bond-equivalent-estimation
description: Use when FT-ICR MS-derived DOM molecular formulae (with elemental compositions
  C, H, O, N, P, S) need per-molecule unsaturation descriptors — DBE, DBEO, DBEAI
  — as input to molecular-trait tables or chemical diversity analyses.
when_to_use_negative:
- Input is peak-level mass spectra (mzML or raw FT-ICR MS peaks) rather than already
  formula-assigned element composition data — molecular formula assignment must precede
  trait calculation
- Analysis target is DOM assembly processes, dark-matter effects, or DOM-microbe networks
  (iDOM's commProc, iDME, H2, iCER functions) rather than basic molecular trait calculation
- The dataset already contains computed DBE/unsaturation descriptors and only other
  traits are missing
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3314
tools:
- name: iDOM
  role: R package whose molTrait function computes DBE, DBEO, and DBEAI from the element
    composition dataset
  repo: https://github.com/jianjunwang/iDOM
- name: devtools
  role: Used to install iDOM from GitHub (devtools::install_github) as a prerequisite
    step
  repo: https://github.com/jianjunwang/iDOM
- name: R
  role: Runtime environment in which the iDOM package and molTrait function are executed
- name: FT-ICR MS
  role: Instrument whose molecular characterization of DOM produces the formula data
    that DBE estimation is applied to
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mlf2.70002
    title: '<i>iDOM</i> : Statistical analysis of dissolved organic matter characterized
      by high‐resolution mass spectrometry'
  source_content_hash: sha256:be005675494ef64895c7b159ef304a1529b71b40e342dde2dddccf317fb9b3e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jianjunwang/iDOM
  tools:
  - FT-ICR MS
  - R
  - devtools
  - iDOM
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# double-bond-equivalent-estimation

## Summary

Calculates double bond equivalents (DBE) and its variants (DBEO, DBEAI) for dissolved organic matter (DOM) molecular formulas as part of the iDOM molTrait function, quantifying the degree of unsaturation of FT-ICR MS-assigned molecular formulae alongside related traits like the aromaticity index (AImod).

## When to use

Use when FT-ICR MS-derived DOM molecular formulae (with elemental compositions C, H, O, N, P, S) need per-molecule unsaturation descriptors — DBE, DBEO, DBEAI — as input to molecular-trait tables or chemical diversity analyses.

## When NOT to use

- Input is peak-level mass spectra (mzML or raw FT-ICR MS peaks) rather than already formula-assigned element composition data — molecular formula assignment must precede trait calculation
- Analysis target is DOM assembly processes, dark-matter effects, or DOM-microbe networks (iDOM's commProc, iDME, H2, iCER functions) rather than basic molecular trait calculation
- The dataset already contains computed DBE/unsaturation descriptors and only other traits are missing

## Inputs

- Element composition table of DOM molecular formulae (e.g., iDOM package example datasets, Tables 1 and S1) with elemental counts per formula
- Molecular intensity dataset from FT-ICR MS characterization distributed with the iDOM package

## Outputs

- Per-formula trait table with DBE, DBEO, and DBEAI columns populated (e.g., mol_traits.csv), alongside the other molTrait outputs (mass, element ratios, AImod, kdefectCH2, GFE, NOSC, Ymet)

## How to apply

Install the iDOM R package from GitHub via devtools::install_github('jianjunwang/iDOM') and load the package's example molecular intensity and element composition datasets (Tables 1 and S1). Run the molTrait function on the element composition data, which computes DBE together with its variants DBEO and DBEAI among the 14 molecular traits (mass, O/C, H/C, N/C, P/C, S/C ratios, kdefectCH2, AImod, GFE, NOSC, Ymet). DBE and its variants serve as unsaturation/aromaticity-related descriptors complementing the modified aromaticity index (AImod), so inspect the returned data frame to confirm the DBE columns are populated for every formula. Write the trait table (e.g., mol_traits.csv) for downstream chemical diversity and dissimilarity analyses.

## Related tools

- **iDOM** (R package whose molTrait function computes DBE, DBEO, and DBEAI from the element composition dataset) — https://github.com/jianjunwang/iDOM
- **devtools** (Used to install iDOM from GitHub (devtools::install_github) as a prerequisite step) — https://github.com/jianjunwang/iDOM
- **R** (Runtime environment in which the iDOM package and molTrait function are executed)
- **FT-ICR MS** (Instrument whose molecular characterization of DOM produces the formula data that DBE estimation is applied to)

## Examples

```
devtools::install_github("jianjunwang/iDOM"); traits <- iDOM::molTrait(element_composition_data)
```

## Evaluation signals

- mol_traits.csv exists, parses as a valid CSV, and contains columns corresponding to all 14 molecular traits named in Table 1, including DBE, DBEO, and DBEAI
- No missing values for DBE columns among formulas present in the input element composition dataset
- DBE values are plausible for DOM formulae (typically small non-negative integers/derived values consistent with C and H counts in the formula)
- AImod and DBEAI columns are coherent with each other, since DBEAI is the DBE variant tied to the aromaticity index

## Limitations

- Requires formula-assigned elemental composition data (C, H, O, N, P, S counts); it cannot be run directly on unassigned peaks or on 'dark matter' (uncharacterized molecules) without prior formula assignment
- The skill is scoped to the iDOM package's basic analysis tier; it does not address assembly-process quantification (commProc), dark-matter interaction effects (iDME), DOM-microbe network specialization (H2), or compositional environmental response (iCER)
- Trait definitions follow iDOM's Table 1 conventions; verify against Table S1 if comparing with traits computed by other pipelines

## Evidence

- [other] The molTrait function calculates molecular chemical characteristics covering mass, element ratios, Kendrick defect (kdefectCH2), AImod, DBE and its variants (DBEO, DBEAI), GFE, NOSC, and carbon use efficiency (Ymet): [evidence span withheld — anchor status: not_found]
- [abstract] iDOM facilitates basic analyses including the calculation of molecular traits: "case studies of basic functions for the calculation of molecular traits"
- [readme] iDOM install instructions from README: "devtools::install_github("jianjunwang/iDOM")"
- [methods] iDOM described as a multifunctional tool for basic analyses such as molecular trait calculation: "iDOM is a multifunctional tool that facilitates basic analyses, such as the calculation of molecular traits, the assignment of molecular classes, and the evaluation of chemical diversity and"
