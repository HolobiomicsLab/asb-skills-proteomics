---
name: replicate-positivity-counting
description: Use when a peptide-by-replicate intensity matrix (e.g. MaxQuant peptides.txt
  with 'Intensity <condition>x' columns for HSILx, ecto Cx, endo Cx) must be annotated
  with per-condition average intensities and positive-replicate counts.
when_to_use_negative:
- The input is already an annotated feature or summary table containing the 'Average
  Intensity_<condition>' and '#Positive Replicates_<condition>' columns
- The intensity matrix has no replicate structure (single column per condition), so
  averaging and positivity counts per condition are undefined
- The goal is raw MALDI imaging peak identification rather than annotating LC-MS/MS
  discovery data for use with MALDImID
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: MALDImID
  role: R package in which the per-condition intensity averaging and positive-replicate
    counting are implemented to exploit LC-MS/MS-based proteomic biomarker discovery
    data
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language used to read peptides.txt (read.delim), grep the condition replicate
    columns, compute rowMeans and positivity counts, and write peptides_annotated.csv
- name: Shiny
  role: Shiny app front-end through which the MALDImID workflow, including the intensity
    summary annotation, can be run interactively
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:96c8416e8d9d5409011f4a732970814e3a6740e19bf5f613e159b650decd82db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# replicate-positivity-counting

## Summary

Annotates a MaxQuant peptides.txt-derived table with per-condition peptide intensity statistics — row-wise replicate averages and counts of positive (non-zero) replicates — as implemented in the MALDImID R/Shiny workflow for LC-MS/MS-based proteomic biomarker discovery.

## When to use

Use when a peptide-by-replicate intensity matrix (e.g. MaxQuant peptides.txt with 'Intensity <condition>x' columns for HSILx, ecto Cx, endo Cx) must be annotated with per-condition average intensities and positive-replicate counts.

## When NOT to use

- The input is already an annotated feature or summary table containing the 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' columns
- The intensity matrix has no replicate structure (single column per condition), so averaging and positivity counts per condition are undefined
- The goal is raw MALDI imaging peak identification rather than annotating LC-MS/MS discovery data for use with MALDImID

## Inputs

- MaxQuant peptides.txt file loaded as an R data frame of peptide rows by replicate intensity columns (Intensity HSILx, Intensity ecto Cx, Intensity endo Cx replicate series)

## Outputs

- Annotated peptide table (peptides_annotated.csv) with six appended columns: 'Average Intensity_HSIL', '#Positive Replicates_HSIL', 'Average Intensity_ecto Cx', '#Positive Replicates_ecto Cx', 'Average Intensity_endo Cx', '#Positive Replicates_endo Cx'

## How to apply

Load peptides.txt into R with read.delim as the peptide-by-replicate intensity matrix. Detect the replicate columns per condition by grep/regex on the 'Intensity <condition>x' naming pattern for the three conditions HSIL, ecto Cx, and endo Cx. For each condition, compute the row-wise mean across its five replicate columns and the row-wise count of replicates with intensity > 0. Append the six summary columns using the Table 3 naming convention ('Average Intensity_<condition>' and '#Positive Replicates_<condition>') and write the annotated table to peptides_annotated.csv. The rationale is to quantify peptide detection reliability across replicates before feeding the LC-MS/MS biomarker discovery results into MALDImID peak identification for MALDI imaging.

## Related tools

- **MALDImID** (R package in which the per-condition intensity averaging and positive-replicate counting are implemented to exploit LC-MS/MS-based proteomic biomarker discovery data) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language used to read peptides.txt (read.delim), grep the condition replicate columns, compute rowMeans and positivity counts, and write peptides_annotated.csv)
- **Shiny** (Shiny app front-end through which the MALDImID workflow, including the intensity summary annotation, can be run interactively) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
peptides$`Average Intensity_HSIL` <- rowMeans(peptides[, grep("Intensity HSIL", names(peptides))]); peptides$`#Positive Replicates_HSIL` <- rowSums(peptides[, grep("Intensity HSIL", names(peptides))] > 0)
```

## Evaluation signals

- The output table contains exactly six appended columns named 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for HSIL, ecto Cx, and endo Cx
- Every '#Positive Replicates_<condition>' value is an integer between 0 and the number of replicate columns detected for that condition (five per condition)
- 'Average Intensity_<condition>' values equal the row-wise mean of that condition's replicate intensity columns and are non-negative
- Row count of peptides_annotated.csv matches the row count of the filtered input peptide table (no rows added or dropped by the annotation step)
- Condition replicate columns are correctly detected by the 'Intensity <condition>x' grep pattern — spot-check that no unrelated 'Intensity' columns are aggregated

## Limitations

- The positivity threshold is fixed at intensity > 0; no other detection cutoff (e.g. S/N or intensity floor) is applied by this step
- Column detection depends on MaxQuant's 'Intensity <condition>x' naming convention; renamed or reformatted intensity columns will be missed by the grep pattern
- The step was designed for the three conditions HSIL, ecto Cx, and endo Cx with five replicates each; other experimental designs require adjusting the column detection logic

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid"
