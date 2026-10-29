---
name: mass-spectrometry-charge-state-handling
description: Use when you have a list of monoisotopic [M+H]+ m/z peaks from positive-mode
  MALDI imaging and an LC-MS/MS peptide mass table (e.g., peptides.txt) and must correlate
  peaks to identified peptides by mass.
when_to_use_negative:
- Input peaks are already neutral masses or de-isotoped/de-protonated values — subtracting
  1 Da would double-correct
- Matching requires higher precision than a ±0.2 Da window (e.g., high-resolution
  ppm-level matching or non-protonated ion species such as [M+Na]+)
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MALDImID
  role: R package implementing the m/z − 1 proton correction and ±0.2 Da tolerance
    mass correlation between MALDI imaging peaks and LC-MS/MS peptides
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language in which the mass-correlation workflow (Tasks 3-4) is implemented
  repo: https://biosciences.shinyapps.io/Maldimid
- name: Shiny
  role: Web app front end exposing the MALDImID peak-identification workflow
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:dadcb90b85afab51242e7f51aedeb96220009f6199f2ec6b7c6222afd1a00c0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mass-spectrometry-charge-state-handling

## Summary

Converts MALDI imaging monoisotopic [M+H]+ m/z peak values to neutral peptide masses by subtracting 1 Da (the proton/cation hydrogen) and matches them against LC-MS/MS peptide masses within a ±0.2 Da tolerance, as implemented in the MALDImID R package/Shiny app.

## When to use

Use when you have a list of monoisotopic [M+H]+ m/z peaks from positive-mode MALDI imaging and an LC-MS/MS peptide mass table (e.g., peptides.txt) and must correlate peaks to identified peptides by mass.

## When NOT to use

- Input peaks are already neutral masses or de-isotoped/de-protonated values — subtracting 1 Da would double-correct
- Matching requires higher precision than a ±0.2 Da window (e.g., high-resolution ppm-level matching or non-protonated ion species such as [M+Na]+)

## Inputs

- MALDI imaging peak list of monoisotopic [M+H]+ m/z values (positive mode)
- LC-MS/MS peptide mass table (peptides.txt) from proteomic biomarker discovery data

## Outputs

- matched_masses.csv: table with MALDI m/z, converted neutral mass, matched LC-MS peptide mass, and mass-difference column

## How to apply

Load the MALDI imaging peak list of monoisotopic [M+H]+ m/z values (positive mode) and the LC-MS peptide mass table from peptides.txt into R. For each MALDI m/z, convert to a neutral mass by subtracting 1 Da (proton mass correction), as done in Tasks 3-4 of the MALDImID R script — e.g., MALDI m/z 865.39 becomes 864.39. Search the LC-MS peptide masses and retain every match falling within a ±0.2 Da tolerance window. Assemble the matched entries (MALDI m/z, neutral mass, matched LC-MS peptide mass, mass difference) into a table and write it to matched_masses.csv. The rationale is that MALDI detects the protonated ion, so the neutral peptide mass used for LC-MS/MS comparison requires removing the cation's hydrogen mass.

## Related tools

- **MALDImID** (R package implementing the m/z − 1 proton correction and ±0.2 Da tolerance mass correlation between MALDI imaging peaks and LC-MS/MS peptides) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language in which the mass-correlation workflow (Tasks 3-4) is implemented) — https://biosciences.shinyapps.io/Maldimid
- **Shiny** (Web app front end exposing the MALDImID peak-identification workflow) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
Rscript -e 'neutral <- peaks$mz - 1; hits <- pep[abs(pep$mass - neutral) <= 0.2, ]; write.csv(hits, "matched_masses.csv")'
```

## Evaluation signals

- Every reported match has |MALDI neutral mass − LC-MS peptide mass| ≤ 0.2 Da
- matched_masses.csv exists and contains the required mass-difference column
- Spot-check conversion: a MALDI m/z of 865.39 must yield a neutral mass of 864.39 before matching
- All matched peptide masses originate from the peptides.txt LC-MS table, not fabricated values

## Limitations

- Fixed ±0.2 Da tolerance and 1 Da proton subtraction assume singly charged [M+H]+ positive-mode peaks; other adducts or charge states are not handled by this skill
- Matches within ±0.2 Da may be ambiguous — multiple peptides can fall inside the window for a single MALDI peak

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [other] MALDImID computes the MALDI peptide mass by subtracting 1 from the observed m/z (the hydrogen of the cation) and then matches LC-MS peptide masses within a ±0.2 Da tolerance window: [evidence span withheld — anchor status: not_found]
