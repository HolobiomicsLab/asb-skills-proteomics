---
name: lcms-peptide-mass-table-parsing
description: Use when a MALDI-to-LC-MS matching step needs a reference table of LC-MS/MS
  peptide masses (e.g., peptides.txt) and the table must be read and formatted in
  R before tolerance-based mass correlation.
when_to_use_negative:
- Input is MALDI imaging peak data rather than the LC-MS/MS peptide mass reference
  table — that data follows a different path (peak picking and proton-mass conversion)
  in the MALDImID workflow
- A matched_masses.csv output table already exists and only validation of ±0.2 Da
  mass differences is required
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MALDImID
  role: R package whose Tasks 3-4 load the LC-MS peptide mass table (peptides.txt)
    and use it for mass correlation against MALDI imaging peaks
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language used to load and parse the peptides.txt peptide mass table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:686337dbadab946f386c94087da443966766f199cc52a75a716a5cdffe0faefb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lcms-peptide-mass-table-parsing

## Summary

Loads and parses the LC-MS/MS-derived peptide mass table (peptides.txt) so that MALDI imaging m/z peaks can be matched against experimentally identified peptide masses within the MALDImID workflow. This parsing step supplies the reference mass list that the MALDImID R package and Shiny app exploit for more specific identification of MALDI imaging proteolytic peaks.

## When to use

Use when a MALDI-to-LC-MS matching step needs a reference table of LC-MS/MS peptide masses (e.g., peptides.txt) and the table must be read and formatted in R before tolerance-based mass correlation.

## When NOT to use

- Input is MALDI imaging peak data rather than the LC-MS/MS peptide mass reference table — that data follows a different path (peak picking and proton-mass conversion) in the MALDImID workflow
- A matched_masses.csv output table already exists and only validation of ±0.2 Da mass differences is required

## Inputs

- peptides.txt: tabular file of peptide masses identified by LC-MS/MS proteomic biomarker discovery data

## Outputs

- In-memory R table of LC-MS peptide masses, ready to be searched within a ±0.2 Da tolerance window by the MALDImID matching step

## How to apply

Read the LC-MS peptide mass table from peptides.txt using R, as done in the MALDImID workflow before the mass-correlation step (Tasks 3-4). Ensure the parsed table contains the peptide masses identified by LC-MS/MS that will serve as reference values for matching against MALDI imaging monoisotopic [M+H]+ peaks. Once loaded, the MALDImID script converts each MALDI m/z to a neutral mass by subtracting 1 Da (the proton/cation hydrogen) and retains LC-MS peptide matches within a ±0.2 Da tolerance window. The parsed table should be structured so each entry can be paired with a MALDI m/z, its converted neutral mass, and the resulting mass difference when written to matched_masses.csv.

## Related tools

- **MALDImID** (R package whose Tasks 3-4 load the LC-MS peptide mass table (peptides.txt) and use it for mass correlation against MALDI imaging peaks) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language used to load and parse the peptides.txt peptide mass table)

## Examples

```
peptides <- read.delim("peptides.txt")
```

## Evaluation signals

- The parsed table contains the LC-MS peptide masses from peptides.txt with no dropped or malformed rows
- Every subsequent reported match has an absolute mass difference between the converted MALDI neutral mass (m/z - 1) and a parsed LC-MS peptide mass of at most 0.2 Da
- The matched entries assembled downstream include MALDI m/z, neutral mass, matched LC-MS peptide mass, and mass difference columns
- Downstream output matched_masses.csv exists and contains the required mass-difference column

## Limitations

- Matching tolerance is fixed at ±0.2 Da and assumes MALDI peaks are monoisotopic [M+H]+ values measured in positive mode; data outside these assumptions will mismatch
- The approach identifies only MALDI peaks whose masses correspond to peptides present in the LC-MS/MS discovery data; unmatched peaks yield no identification

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] MALDI peaks identified via LC-MS reference table: [evidence span withheld — anchor status: not_found]
- [abstract] shiny app availability: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
