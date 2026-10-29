---
name: proteomic-biomarker-peak-annotation
description: Use when you have a list of monoisotopic MALDI imaging [M+H]+ m/z peaks
  (positive mode) and an LC-MS/MS peptide mass table (e.g., peptides.txt) and need
  to identify which imaging peaks correspond to proteomically observed peptides.
when_to_use_negative:
- Input peaks are in negative ion mode or are not monoisotopic [M+H]+ values — the
  proton subtraction of 1 Da and matching logic assume positive-mode protonated peptides.
- You need de novo or database-driven peptide identification from MALDI spectra rather
  than annotation against an existing LC-MS/MS biomarker discovery table.
- Your mass tolerance requirements differ substantially (e.g., high-resolution FT
  data needing ppm-level matching) — the ±0.2 Da window is specific to this MALDI-to-LC-MS
  matching step.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3522
tools:
- name: MALDImID
  role: R package implementing the MALDI-to-LC-MS peptide mass matching with proton-mass
    correction and ±0.2 Da tolerance
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Runtime environment for the MALDImID package and matching workflow
- name: Shiny
  role: Web interface for MALDImID, usable directly at the hosted app address
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:03b0a09cbcc3c4ffd997e81697aa191280df4ae9244d13bb68cfe1fb044d533c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# proteomic-biomarker-peak-annotation

## Summary

Annotate MALDI imaging proteolytic peaks by matching observed [M+H]+ m/z values against LC-MS/MS-based proteomic biomarker discovery data, as implemented in the MALDImID R package and Shiny app.

## When to use

Use when you have a list of monoisotopic MALDI imaging [M+H]+ m/z peaks (positive mode) and an LC-MS/MS peptide mass table (e.g., peptides.txt) and need to identify which imaging peaks correspond to proteomically observed peptides.

## When NOT to use

- Input peaks are in negative ion mode or are not monoisotopic [M+H]+ values — the proton subtraction of 1 Da and matching logic assume positive-mode protonated peptides.
- You need de novo or database-driven peptide identification from MALDI spectra rather than annotation against an existing LC-MS/MS biomarker discovery table.
- Your mass tolerance requirements differ substantially (e.g., high-resolution FT data needing ppm-level matching) — the ±0.2 Da window is specific to this MALDI-to-LC-MS matching step.

## Inputs

- MALDI imaging peak list of monoisotopic [M+H]+ m/z values (positive mode)
- LC-MS/MS peptide mass table (peptides.txt) from proteomic biomarker discovery data

## Outputs

- matched_masses.csv: table with MALDI m/z, converted neutral mass, matched LC-MS peptide mass, and mass difference columns

## How to apply

Load the MALDI imaging peak list of monoisotopic [M+H]+ m/z values (positive mode) and the LC-MS peptide mass table from peptides.txt into R. Convert each MALDI m/z to a neutral mass by subtracting 1 Da to correct for the proton of the cation (e.g., MALDI m/z 865.39 gives mass 864.39). Search the LC-MS peptide masses and retain all matches falling within a ±0.2 Da tolerance window around each neutral mass. Assemble the matched entries (MALDI m/z, neutral mass, matched LC-MS peptide mass, mass difference) into a table and write it to matched_masses.csv. The rationale is that the neutral peptide mass, not the protonated mass, should be compared across the two modalities, and the ±0.2 Da window accommodates the lower mass accuracy of MALDI imaging instruments.

## Related tools

- **MALDImID** (R package implementing the MALDI-to-LC-MS peptide mass matching with proton-mass correction and ±0.2 Da tolerance) — https://biosciences.shinyapps.io/Maldimid
- **R** (Runtime environment for the MALDImID package and matching workflow)
- **Shiny** (Web interface for MALDImID, usable directly at the hosted app address) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
Rscript -e 'mz <- 865.39 - 1; matches <- subset(peptides, abs(peptide_mass - mz) <= 0.2); write.csv(matches, "matched_masses.csv")'
```

## Evaluation signals

- Every reported match has an absolute mass difference between the converted MALDI neutral mass and the LC-MS peptide mass of at most 0.2 Da.
- matched_masses.csv exists and contains the required mass-difference column.
- Neutral masses equal the input MALDI m/z minus exactly 1 Da (spot-check: 865.39 → 864.39).
- Each matched LC-MS peptide mass originates from the peptides.txt input table.
- All input MALDI m/z values either appear in the output (matched) or can be accounted for as unmatched, with no duplicated or dropped peak entries.

## Limitations

- The ±0.2 Da tolerance is a fixed window; it may admit false-positive matches at low mass accuracy or miss true matches if calibration is poor.
- The method annotates peaks only against peptides present in the supplied LC-MS/MS biomarker discovery table — peptides absent from that table cannot be identified.
- Matching assumes positive-mode monoisotopic [M+H]+ observations; other adducts or charge states are not handled by the 1 Da proton subtraction.

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [other] MALDImID computes the MALDI peptide mass by subtracting 1 from the observed m/z (the hydrogen of the cation) and then matches LC-MS peptide masses within a ±0.2 Da tolerance window, e.g., MALDI m/z 865.39 gives mass 864.39 ± 0.2.: [evidence span withheld — anchor status: not_found]
- [abstract] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid.: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [abstract] The package is made available under the GPL 3 license.: "The package is made available under the GPL 3 license."
