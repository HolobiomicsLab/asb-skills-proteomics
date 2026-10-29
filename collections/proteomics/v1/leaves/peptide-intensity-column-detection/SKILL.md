---
name: peptide-intensity-column-detection
description: Use when a MaxQuant peptides.txt (or similar peptide-by-replicate intensity
  matrix) must be annotated with 'Average Intensity_<condition>' and '#Positive Replicates_<condition>'
  columns for conditions such as HSIL, ecto Cx, and endo Cx.
when_to_use_negative:
- Input is not a MaxQuant-style peptides.txt with replicate 'Intensity <condition>x'
  columns, so the grep-based column detection cannot find condition groups.
- The analysis needs raw replicate intensities rather than per-condition summaries;
  this skill only appends aggregated average and positive-replicate columns.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MALDImID
  role: R package and Shiny app whose peptide annotation workflow appends per-condition
    average intensity and positive-replicate count columns to the filtered peptide
    table
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language used to read peptides.txt with read.delim, grep the condition replicate
    columns, and compute row-wise averages and positive-replicate counts
- name: Shiny
  role: Web interface through which the MALDImID annotation workflow can be run directly
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:2934197b4e23f1ab3cefb93590dfbe15e9e6e44e2759ab47cad7092af058f643
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# peptide-intensity-column-detection

## Summary

Detects condition-specific replicate intensity columns in a MaxQuant peptides.txt file (e.g. 'Intensity <condition>x' naming for HSIL, ecto Cx, endo Cx) and appends per-condition row-wise average intensities and positive-replicate counts to the MALDImID peptide table. This connects LC-MS/MS biomarker discovery data to the per-peptide summary columns used downstream in MALDImID.

## When to use

Use when a MaxQuant peptides.txt (or similar peptide-by-replicate intensity matrix) must be annotated with 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' columns for conditions such as HSIL, ecto Cx, and endo Cx.

## When NOT to use

- Input is not a MaxQuant-style peptides.txt with replicate 'Intensity <condition>x' columns, so the grep-based column detection cannot find condition groups.
- The analysis needs raw replicate intensities rather than per-condition summaries; this skill only appends aggregated average and positive-replicate columns.

## Inputs

- MaxQuant peptides.txt file (peptide-by-replicate intensity matrix, tab-delimited)
- Condition names / replicate column naming pattern ('Intensity <condition>x' for HSIL, ecto Cx, endo Cx)

## Outputs

- Annotated peptide table (peptides_annotated.csv) with appended 'Average Intensity_<condition>' columns
- Appended '#Positive Replicates_<condition>' integer count columns for each of the three conditions

## How to apply

Load peptides.txt into R with read.delim so each row is a peptide and each column a replicate intensity. Use grep/regex to detect the replicate columns matching the 'Intensity <condition>x' naming pattern for the three conditions HSIL, ecto Cx, and endo Cx (five replicates each). For each condition, compute the row-wise mean across its replicate intensity columns and the row-wise count of replicates with intensity > 0. Append these as six new columns named 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' per the Table 3 convention, then write the annotated table to peptides_annotated.csv as delimited text. The rationale is that MALDImID exploits LC-MS/MS-based proteomic biomarker discovery data to make peak identification in bottom-up MALDI imaging more specific, so per-condition intensity summaries must be consistently named and computable per peptide row.

## Related tools

- **MALDImID** (R package and Shiny app whose peptide annotation workflow appends per-condition average intensity and positive-replicate count columns to the filtered peptide table) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language used to read peptides.txt with read.delim, grep the condition replicate columns, and compute row-wise averages and positive-replicate counts)
- **Shiny** (Web interface through which the MALDImID annotation workflow can be run directly) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
peptides <- read.delim("peptides.txt"); cols <- grep("Intensity HSILx", names(peptides)); peptides$`Average Intensity_HSIL` <- rowMeans(peptides[, cols])
```

## Evaluation signals

- Output table contains exactly six appended summary columns: 'Average Intensity_<condition>' and '#Positive Replicates_<condition>' for HSIL, ecto Cx, and endo Cx.
- Every '#Positive Replicates_<condition>' value is an integer between 0 and the number of replicate columns (five) for that condition.
- Each 'Average Intensity_<condition>' value equals the row-wise mean of that condition's replicate intensity columns for the same peptide row.
- Column detection via the 'Intensity <condition>x' regex matches exactly five replicate columns per condition before aggregation.
- peptides_annotated.csv is written as delimited text and row count matches the filtered input peptide table.

## Limitations

- Column detection depends on the MaxQuant 'Intensity <condition>x' naming convention; renamed or differently structured intensity columns will not be matched.
- The skill assumes exactly five replicates per condition and the three conditions HSIL, ecto Cx, and endo Cx; other experimental designs require adapting the condition list.
- Positive-replicate counting uses a strict intensity > 0 rule, which is sensitive to how MaxQuant encodes missing or zero intensities.

## Evidence

- [abstract] R package and Shiny app for peptide intensity annotation: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] Purpose: more specific identification of MALDI imaging peaks: [evidence span withheld — anchor status: not_found]
- [abstract] Shiny app deployment address: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
