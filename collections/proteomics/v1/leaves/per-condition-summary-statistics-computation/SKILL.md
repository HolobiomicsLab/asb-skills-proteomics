---
name: per-condition-summary-statistics-computation
description: Use when a MaxQuant peptides.txt table contains replicate intensity columns
  named 'Intensity <condition>x' (e.g. HSILx, ecto Cx, endo Cx) and per-condition
  average intensity and positive-replicate counts must be appended for downstream
  MALDImID peak identification.
when_to_use_negative:
- Input is not a MaxQuant peptides.txt peptide-level table with replicate 'Intensity
  <condition>x' columns (e.g. it is already a summarized feature/protein table with
  per-condition values).
- The analysis goal is MALDI imaging peak detection itself rather than summarizing
  LC-MS/MS discovery-data replicate intensities feeding into MALDImID.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MALDImID
  role: R package / Shiny app that exploits LC-MS/MS-based proteomic biomarker discovery
    data; the per-condition summary columns are appended within this workflow for
    specific identification of MALDI imaging proteolytic peaks
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: 'Implementation language: read.delim to load peptides.txt, grep/regex to detect
    condition replicate columns, rowMeans and row sums of intensity > 0 to compute
    summaries'
- name: MALDImIDshiny
  role: Shiny app front end of the MALDImID workflow in which the annotated peptide
    table is used
  repo: https://github.com/budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:75bd7835aebb5eb2a5702476282e2671bfb39f0fcb27b7fd6e5e9fe2dea88548
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - MALDImIDshiny
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# per-condition-summary-statistics-computation

## Summary

Computation of per-condition peptide intensity summaries — row-wise mean intensity and positive-replicate counts — from MaxQuant peptides.txt replicate columns, as implemented in the MALDImID R package/Shiny app for LC-MS/MS-based proteomic biomarker discovery feeding MALDI imaging peak identification.

## When to use

Use when a MaxQuant peptides.txt table contains replicate intensity columns named 'Intensity <condition>x' (e.g. HSILx, ecto Cx, endo Cx) and per-condition average intensity and positive-replicate counts must be appended for downstream MALDImID peak identification.

## When NOT to use

- Input is not a MaxQuant peptides.txt peptide-level table with replicate 'Intensity <condition>x' columns (e.g. it is already a summarized feature/protein table with per-condition values).
- The analysis goal is MALDI imaging peak detection itself rather than summarizing LC-MS/MS discovery-data replicate intensities feeding into MALDImID.

## Inputs

- MaxQuant peptides.txt file (peptide-by-replicate intensity matrix, tab-delimited)
- Condition names to match: 'HSILx', 'ecto Cx', 'endo Cx' (five replicate intensity columns per condition)

## Outputs

- peptides_annotated.csv — filtered peptide table with six appended summary columns: 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for HSIL, ecto Cx, and endo Cx

## How to apply

Load peptides.txt into R with read.delim so each row is a peptide and columns hold replicate intensities. Use grep/regex on the 'Intensity <condition>x' naming pattern to detect the five replicate columns for each of the three conditions (HSIL, ecto Cx, endo Cx). For each condition compute the row-wise mean across its replicate columns and the row-wise count of replicates with intensity > 0. Append these as 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' columns following the Table 3 naming convention, then write the annotated table to peptides_annotated.csv. The rationale is that averaging replicates and tracking positivity per condition summarizes LC-MS/MS proteomic biomarker discovery evidence so MALDImID can more specifically identify peaks in bottom-up MALDI imaging data.

## Related tools

- **MALDImID** (R package / Shiny app that exploits LC-MS/MS-based proteomic biomarker discovery data; the per-condition summary columns are appended within this workflow for specific identification of MALDI imaging proteolytic peaks) — https://biosciences.shinyapps.io/Maldimid
- **R** (Implementation language: read.delim to load peptides.txt, grep/regex to detect condition replicate columns, rowMeans and row sums of intensity > 0 to compute summaries)
- **MALDImIDshiny** (Shiny app front end of the MALDImID workflow in which the annotated peptide table is used) — https://github.com/budczies/MALDImIDshiny

## Examples

```
pep <- read.delim("peptides.txt"); for (c in c("HSILx","ecto Cx","endo Cx")) { cols <- grep(paste0("Intensity ", c), names(pep)); pep[[paste0("Average Intensity_",c)]] <- rowMeans(pep[,cols]); pep[[paste0("#Positive Replicates_",c)]] <- rowSums(pep[,cols] > 0) }; write.csv(pep, "peptides_annotated.csv")
```

## Evaluation signals

- Output table contains exactly six appended summary columns: 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for all three conditions (HSIL, ecto Cx, endo Cx).
- Every '#Positive Replicates_<condition>' value is an integer between 0 and the number of replicates (5) for that condition.
- 'Average Intensity_<condition>' equals rowMeans over the grep-detected 'Intensity <condition>x' columns and is >= 0.
- Row count of peptides_annotated.csv matches the filtered peptide table; no peptide rows are dropped or duplicated by the annotation step.
- Recomputing summaries independently in R reproduces the appended columns to floating-point tolerance.

## Limitations

- Column detection depends on the exact MaxQuant 'Intensity <condition>x' naming pattern; renamed or differently numbered replicate columns will not be matched by the grep/regex step.
- The positive-replicate definition (intensity > 0) is a simple positivity count, not a statistical replicate-quality criterion; noise-only nonzero intensities still count as positive.
- The condition set is specific to the study design (HSIL, ecto Cx, endo Cx with five replicates each); other datasets require adapting the condition list.

## Evidence

- [abstract] MALDImID implemented for LC-MS/MS biomarker discovery feeding MALDI imaging: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [abstract] MALDImID purpose in abstract: specific identification of peaks in bottom-up MALDI imaging data: [evidence span withheld — anchor status: not_found]
- [abstract] Shiny app availability URL (repo/deployment grounding): "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid"
