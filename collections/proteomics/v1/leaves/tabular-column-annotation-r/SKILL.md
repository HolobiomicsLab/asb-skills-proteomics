---
name: tabular-column-annotation-r
description: Use when a MaxQuant peptides.txt (or similar peptide-by-replicate intensity
  matrix) must be annotated with per-condition aggregate columns such as 'Average
  Intensity_<condition>' and '#Positive Replicates_<condition>'.
when_to_use_negative:
- The input table already contains the per-condition average and positive-replicate
  columns, so re-annotation would duplicate columns.
- The intensity file is not a peptide-level replicate matrix with 'Intensity <condition>x'
  column names (e.g. it is a feature-level or proteingroup-level table), so grep-based
  condition detection would fail.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language used to load peptides.txt (read.delim), grep the condition replicate
    columns, compute row-wise means and positive-replicate counts, and write peptides_annotated.csv
- name: MALDImID
  role: R package and Shiny app whose workflow appends the per-condition intensity
    summary columns to the peptide table
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:cc7b7be5f6ef258428ef535362c6ac98256162cdeda54be11b69360992dda627
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# tabular-column-annotation-r

## Summary

An R-based skill for appending per-condition summary columns (row-wise replicate means and positive-replicate counts) to a MaxQuant peptides.txt intensity table, as implemented in the MALDImID R package/Shiny app for LC-MS/MS proteomic biomarker discovery data.

## When to use

Use when a MaxQuant peptides.txt (or similar peptide-by-replicate intensity matrix) must be annotated with per-condition aggregate columns such as 'Average Intensity_<condition>' and '#Positive Replicates_<condition>'.

## When NOT to use

- The input table already contains the per-condition average and positive-replicate columns, so re-annotation would duplicate columns.
- The intensity file is not a peptide-level replicate matrix with 'Intensity <condition>x' column names (e.g. it is a feature-level or proteingroup-level table), so grep-based condition detection would fail.

## Inputs

- MaxQuant peptides.txt peptide-by-replicate intensity matrix (tab-delimited, read via read.delim)
- Condition naming pattern 'Intensity <condition>x' for conditions HSIL, ecto Cx, and endo Cx (five replicate columns each)

## Outputs

- peptides_annotated.csv: the filtered peptide table with six appended summary columns ('Average Intensity_HSIL', '#Positive Replicates_HSIL', 'Average Intensity_ecto Cx', '#Positive Replicates_ecto Cx', 'Average Intensity_endo Cx', '#Positive Replicates_endo Cx')

## How to apply

Load peptides.txt into R with read.delim so each row is a peptide and columns are replicate intensities. Use grep with the 'Intensity <condition>x' naming pattern to detect the five replicate columns for each of the three conditions ('HSILx', 'ecto Cx', 'endo Cx'). For each condition, compute rowMeans over its replicate columns to create an 'Average Intensity_<condition>' column, and compute a row-wise count of replicates with intensity > 0 to create a '#Positive Replicates_<condition>' column, following the Table 3 naming convention. Append all six summary columns to the filtered peptide table and write the result to disk as a delimited text file (peptides_annotated.csv). The rationale is that condition-level peptide summaries feed MALDImID's more specific identification of MALDI imaging proteolytic peaks from LC-MS/MS-based proteomic biomarker discovery data.

## Related tools

- **R** (Language used to load peptides.txt (read.delim), grep the condition replicate columns, compute row-wise means and positive-replicate counts, and write peptides_annotated.csv)
- **MALDImID** (R package and Shiny app whose workflow appends the per-condition intensity summary columns to the peptide table) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
peps <- read.delim("peptides.txt"); cols <- grep("Intensity HSILx", names(peps)); peps$`Average Intensity_HSIL` <- rowMeans(peps[, cols]); peps$`#Positive Replicates_HSIL` <- rowSums(peps[, cols] > 0)
```

## Evaluation signals

- The output table contains exactly six appended summary columns named per the Table 3 convention: 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for HSIL, ecto Cx, and endo Cx.
- Every '#Positive Replicates_<condition>' value is an integer between 0 and the number of replicate columns (five) for that condition.
- 'Average Intensity_<condition>' values equal the row-wise mean of the condition's five replicate intensity columns and are >= 0.
- Row count of peptides_annotated.csv matches the filtered input peptide table (no rows added or dropped by the annotation step).

## Limitations

- Condition detection depends on the exact MaxQuant 'Intensity <condition>x' column naming convention; deviations in condition names break the grep pattern.
- The skill is specific to the three conditions used in the MALDImID study (HSIL, ecto Cx, endo Cx) and the Table 3 naming convention; other studies must adapt the condition list and column names.

## Evidence

- [abstract] R package and Shiny app implementation for proteomic biomarker discovery: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] MALDImID package license: "The package is made available under the GPL 3 license."
