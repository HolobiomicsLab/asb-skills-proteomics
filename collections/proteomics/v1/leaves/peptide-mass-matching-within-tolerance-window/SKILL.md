---
name: peptide-mass-matching-within-tolerance-window
description: Use when you have a list of monoisotopic positive-mode MALDI imaging
  [M+H]+ m/z peaks and an LC-MS/MS peptide mass table (e.g., peptides.txt) and need
  to identify which proteolytic peptides the MALDI peaks correspond to.
when_to_use_negative:
- Input m/z values are already neutral peptide masses — applying the 1 Da proton subtraction
  would systematically shift all matches
- Masses are in negative mode or not monoisotopic [M+H]+ values — the proton-cation
  correction of 1 Da would be incorrect
- ppm-level accuracy matching is required (e.g., high-resolution MALDI) — the ±0.2
  Da window is designed for MALDI mass accuracy and would be too coarse for a ppm
  criterion
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MALDImID
  role: R package implementing the MALDI m/z-to-neutral-mass conversion and ±0.2 Da
    tolerance matching against LC-MS/MS peptide masses
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language in which the mass correlation workflow (load peak list, subtract
    1 Da, filter by tolerance, write matched_masses.csv) is executed
- name: Shiny
  role: Web app providing direct browser access to MALDImID without local installation
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:912bf0f1ddc7caedee1735f016115822a13ec687618120d9666f6bdefe576a58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# peptide-mass-matching-within-tolerance-window

## Summary

Correlates MALDI imaging [M+H]+ m/z peaks with LC-MS/MS peptide masses by converting MALDI m/z to neutral mass (subtracting the 1 Da proton) and matching within a ±0.2 Da tolerance, as implemented in the MALDImID R package.

## When to use

Use when you have a list of monoisotopic positive-mode MALDI imaging [M+H]+ m/z peaks and an LC-MS/MS peptide mass table (e.g., peptides.txt) and need to identify which proteolytic peptides the MALDI peaks correspond to.

## When NOT to use

- Input m/z values are already neutral peptide masses — applying the 1 Da proton subtraction would systematically shift all matches
- Masses are in negative mode or not monoisotopic [M+H]+ values — the proton-cation correction of 1 Da would be incorrect
- ppm-level accuracy matching is required (e.g., high-resolution MALDI) — the ±0.2 Da window is designed for MALDI mass accuracy and would be too coarse for a ppm criterion

## Inputs

- List of monoisotopic positive-mode MALDI imaging [M+H]+ m/z peaks of interest
- LC-MS peptide mass table from peptides.txt (peptide masses identified by LC-MS/MS)

## Outputs

- matched_masses.csv — table with columns: MALDI m/z, converted neutral mass, matched LC-MS peptide mass, mass difference

## How to apply

Load the MALDI imaging peak list of monoisotopic [M+H]+ m/z values and the LC-MS peptide mass table from peptides.txt in R. Convert each MALDI m/z to a neutral mass by subtracting 1 Da (the hydrogen of the cation), as implemented in Tasks 3-4 of the MALDImID R script — e.g., MALDI m/z 865.39 becomes neutral mass 864.39. For each neutral mass, search the LC-MS peptide masses and retain all matches within a ±0.2 Da tolerance window. Assemble matched entries (MALDI m/z, neutral mass, matched LC-MS peptide mass, mass difference) into a table and write it to matched_masses.csv. The rationale is that MALDI measures protonated ions, so the observed m/z must be proton-corrected before comparing against neutral peptide masses from LC-MS/MS proteomic biomarker discovery data.

## Related tools

- **MALDImID** (R package implementing the MALDI m/z-to-neutral-mass conversion and ±0.2 Da tolerance matching against LC-MS/MS peptide masses) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language in which the mass correlation workflow (load peak list, subtract 1 Da, filter by tolerance, write matched_masses.csv) is executed)
- **Shiny** (Web app providing direct browser access to MALDImID without local installation) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
peaks <- read.table('peaks.txt'); peptides <- read.delim('peptides.txt'); m <- peptides[abs(peptides$mass - (peaks$mz - 1)) <= 0.2, ]; write.csv(m, 'matched_masses.csv')
```

## Evaluation signals

- Every reported match has |converted MALDI neutral mass − LC-MS peptide mass| ≤ 0.2 Da
- matched_masses.csv exists and contains the required mass-difference column
- Spot-check a known conversion: MALDI m/z 865.39 must yield neutral mass 864.39 before tolerance matching
- No match is reported for an m/z peak whose ±0.2 Da window contains no peptide mass in peptides.txt

## Limitations

- The fixed ±0.2 Da tolerance reflects MALDI mass accuracy; it may yield false matches in dense mass regions and cannot be tightened without instrument-dependent recalibration
- Matching is mass-only (no retention time, fragmentation, or isotopic pattern validation), so assignments are candidate-level, not definitive identifications

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid.: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
