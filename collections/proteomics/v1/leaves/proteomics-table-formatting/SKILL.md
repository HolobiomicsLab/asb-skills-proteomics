---
name: proteomics-table-formatting
description: 'Use when a peptide-by-replicate intensity matrix (e.g. MaxQuant peptides.txt)
  needs per-condition summary columns (average intensity, #Positive Replicates) appended
  before downstream MALDImID analysis.'
when_to_use_negative:
- The input table already contains the per-condition 'Average Intensity_' and '#Positive
  Replicates_' summary columns (re-appending would duplicate them).
- The input is not a peptides.txt-style peptide-by-replicate intensity matrix, e.g.
  it lacks 'Intensity <condition>x' replicate columns for the conditions of interest.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Used to load peptides.txt (read.delim), detect condition replicate columns
    via grep/regex, compute row-wise averages and positive-replicate counts, and write
    peptides_annotated.csv
- name: MALDImID
  role: Downstream R package and Shiny app that consumes the LC-MS/MS-based proteomic
    biomarker discovery data annotated by this formatting step
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:1fab6ca0c069009785404ceba99596ddb34b1c79f4f3c9221b966afe7fe5e286
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# proteomics-table-formatting

## Summary

Annotates a MaxQuant peptides.txt-derived peptide table with per-condition summary columns — row-wise average intensity and counts of positive replicates — for the MALDImID conditions 'HSILx', 'ecto Cx' and 'endo Cx', following the Table 3 naming convention.

## When to use

Use when a peptide-by-replicate intensity matrix (e.g. MaxQuant peptides.txt) needs per-condition summary columns (average intensity, #Positive Replicates) appended before downstream MALDImID analysis.

## When NOT to use

- The input table already contains the per-condition 'Average Intensity_' and '#Positive Replicates_' summary columns (re-appending would duplicate them).
- The input is not a peptides.txt-style peptide-by-replicate intensity matrix, e.g. it lacks 'Intensity <condition>x' replicate columns for the conditions of interest.

## Inputs

- MaxQuant peptides.txt file (peptide-by-replicate intensity matrix) with 'Intensity <condition>x' replicate columns for HSILx, ecto Cx, and endo Cx

## Outputs

- Annotated peptide table (peptides_annotated.csv) with six appended columns: 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for HSIL, ecto Cx, and endo Cx

## How to apply

Load peptides.txt into R with read.delim as the peptide-by-replicate intensity matrix. Detect the replicate columns for each of the three conditions ('HSILx', 'ecto Cx', 'endo Cx') using grep/regex on the 'Intensity <condition>x' naming pattern. For each condition, compute the row-wise average peptide intensity across its five replicate columns and the row-wise count of replicates with intensity > 0. Append these as six columns named 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' per the Table 3 convention, then write the annotated table to disk as peptides_annotated.csv. The rationale is that MALDImID consumes LC-MS/MS-based proteomic biomarker discovery data to make MALDI imaging peak identification more specific, and these per-condition summaries feed that workflow.

## Related tools

- **R** (Used to load peptides.txt (read.delim), detect condition replicate columns via grep/regex, compute row-wise averages and positive-replicate counts, and write peptides_annotated.csv)
- **MALDImID** (Downstream R package and Shiny app that consumes the LC-MS/MS-based proteomic biomarker discovery data annotated by this formatting step) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
pep <- read.delim("peptides.txt"); idx <- grep("^Intensity HSILx|^Intensity ecto Cx|^Intensity endo Cx", names(pep)); pep$`Average Intensity_HSIL` <- rowMeans(pep[, grep("^Intensity HSILx", names(pep))]); write.csv(pep, "peptides_annotated.csv")
```

## Evaluation signals

- The output table contains exactly six appended summary columns: 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for HSIL, ecto Cx, and endo Cx.
- Every '#Positive Replicates_<condition>' value is an integer between 0 and the number of replicates for that condition (expected up to five).
- Each 'Average Intensity_<condition>' column equals the row-wise mean of that condition's replicate intensity columns.
- Each '#Positive Replicates_<condition>' value equals the row-wise count of that condition's replicate columns with intensity > 0.
- Output file peptides_annotated.csv parses as a delimited text table with one row per peptide of the filtered input.

## Limitations

- The column-detection regex depends on the MaxQuant 'Intensity <condition>x' naming pattern; differently named replicate columns for the three conditions will not be detected.
- The procedure is specific to the three conditions HSIL, ecto Cx, and endo Cx and their five replicate columns as used in the MALDImID workflow; other experimental designs require adaptation.
- The public README for budczies/MALDImIDshiny provides no additional usage details, so implementation relies on the article's described workflow.

## Evidence

- [abstract] An R package and Shiny app (MALDImID) were implemented to exploit LC-MS/MS-based proteomic biomarker discovery data for more specific identification of peaks in bottom-up MALDI imaging data: [evidence span withheld — anchor status: not_found]
- [other] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid"
- [abstract] The package is made available under the GPL 3 license: "The package is made available under the GPL 3 license."
