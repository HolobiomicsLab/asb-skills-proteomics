---
name: result-table-reference-comparison
description: Use when reproducing the MALDImID LC-MS/MS-to-MALDI imaging peak correlation
  analysis and you need to verify your regenerated results table against the authors'
  deposited Supplementary data 4 output.
when_to_use_negative:
- When the goal is de novo peak identification rather than validating a reproduced
  MALDImID output against the deposited reference table
- When the input LC-MS/MS proteomic data is not MaxQuant-derived peptides.txt output,
  since the correlation routine expects that format
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MALDImID
  role: R package whose correlation routine generates the results table that is compared
    against the deposited reference
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Runtime environment in which the MALDImID correlation routine is executed
    and the xlsx results table is assembled
- name: Shiny
  role: Web app interface through which the MALDImID analysis can alternatively be
    run before table comparison
  repo: https://biosciences.shinyapps.io/Maldimid
- name: MALDImIDshiny
  role: GitHub repository associated with the Shiny app used to run the analysis
  repo: https://github.com/budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:d94a489006d225b7983780a547faa44886d3a8e7381e313d987b7b5875f8e186
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - MALDImIDshiny
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# result-table-reference-comparison

## Summary

Validation skill for the MALDImID spatialomics workflow: rerun the correlation analysis on the supplied inputs (MaxQuant-exported peptides.txt, HSIL(+) cluster gene-name list, MALDI m/z list) and compare the generated annotated results table row-by-row against the deposited reference output (Supplementary data 4).

## When to use

Use when reproducing the MALDImID LC-MS/MS-to-MALDI imaging peak correlation analysis and you need to verify your regenerated results table against the authors' deposited Supplementary data 4 output.

## When NOT to use

- When the goal is de novo peak identification rather than validating a reproduced MALDImID output against the deposited reference table
- When the input LC-MS/MS proteomic data is not MaxQuant-derived peptides.txt output, since the correlation routine expects that format

## Inputs

- peptides.txt exported from MaxQuant (Supplementary data 1)
- HSIL(+) cluster gene-name list (Supplementary data 2)
- MALDI m/z list of [M+H]+ peaks (Supplementary data 3)

## Outputs

- Annotated MALDImID results table as an xlsx file with 13 columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages, positive replicate counts, MALDI mass, mass difference)
- Row-by-row comparison against the deposited reference table (Supplementary data 4)

## How to apply

Load the three supplied inputs into R: the peptides.txt file exported from MaxQuant (Supplementary data 1), the HSIL(+) cluster gene-name list (Supplementary data 2), and the MALDI m/z list (Supplementary data 3). Execute the MALDImID R package correlation routine to match each MALDI m/z [M+H]+ peak against the LC-MS/MS-based proteomic biomarker discovery data, restricted to the HSIL(+) cluster gene names. Assemble the results table with the 13 documented columns: MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, and mass difference. Export the annotated table as an xlsx file, then validate row-by-row against the deposited reference (Supplementary data 4), checking that MALDI m/z [M+H]+ values, gene names, and mass difference columns match the reference deposit.

## Related tools

- **MALDImID** (R package whose correlation routine generates the results table that is compared against the deposited reference) — https://biosciences.shinyapps.io/Maldimid
- **R** (Runtime environment in which the MALDImID correlation routine is executed and the xlsx results table is assembled)
- **Shiny** (Web app interface through which the MALDImID analysis can alternatively be run before table comparison) — https://biosciences.shinyapps.io/Maldimid
- **MALDImIDshiny** (GitHub repository associated with the Shiny app used to run the analysis) — https://github.com/budczies/MALDImIDshiny

## Evaluation signals

- The produced xlsx table contains all 13 documented columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages, positive replicate counts, MALDI mass, mass difference)
- MALDI m/z [M+H]+ values in the reproduced table match the Supplementary data 4 reference row-by-row
- Gene names match the deposited reference table for every row
- Mass difference column matches the deposited reference table
- Correlation was restricted to HSIL(+) cluster gene names, as specified in the workflow

## Limitations

- Validation depends entirely on availability of the deposited Supplementary data 4 reference table; without it, no row-by-row comparison is possible
- The workflow assumes the exact three supplied inputs (MaxQuant peptides.txt, HSIL(+) gene-name list, MALDI m/z list); other input formats are not covered by the documented reproduction
- The repository README for budczies/MALDImIDshiny provides no additional usage documentation, so invocation details must come from the article

## Evidence

- [abstract] Abstract — R package and Shiny app purpose: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [abstract] Abstract — Shiny app URL: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [other] Source card — results table and Supplementary data 4 deposit: [evidence span withheld — anchor status: not_found]
- [supplementary] Source card — 13-column table schema: [evidence span withheld — anchor status: not_found]
