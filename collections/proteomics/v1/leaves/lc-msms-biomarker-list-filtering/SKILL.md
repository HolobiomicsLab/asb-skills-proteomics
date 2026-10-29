---
name: lc-msms-biomarker-list-filtering
description: Use when MALDI imaging proteolytic peaks (m/z [M+H]+ list) need to be
  annotated against LC-MS/MS proteomic discovery data, and a gene-name cluster list
  must first be used to restrict candidate biomarkers before correlation matching.
when_to_use_negative:
- The input is already an annotated MALDI peak-to-protein mapping table and no gene-name
  restriction is needed
- No LC-MS/MS-based proteomic biomarker discovery data (e.g., MaxQuant peptides.txt)
  is available to correlate against MALDI m/z peaks
- The analysis goal is de novo peptide or protein identification from MALDI spectra
  rather than correlating measured m/z peaks against discovery-data candidates
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: MALDImID
  role: R package whose correlation routine matches MALDI m/z [M+H]+ peaks against
    gene-name-filtered LC-MS/MS proteomic biomarker discovery data
  repo: https://biosciences.shinyapps.io/Maldimid
- name: Shiny
  role: Web-app framework hosting MALDImIDshiny so the same correlation analysis can
    be run interactively without local R installation
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Environment in which the MALDImID package is executed to filter and correlate
    the LC-MS/MS data with MALDI peak lists
  repo: budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:4d6794fc2c36b7e4feeb474b3156b1dfb58bd3b6e2ba4426b946e22648370f77
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# LC-MS/MS biomarker list filtering

## Summary

Restricts LC-MS/MS-based proteomic biomarker discovery data to a gene-name list of interest (e.g., the HSIL(+) cluster) so that MALDI imaging [M+H]+ peaks can be matched only against candidate biomarkers, using the MALDImID R package/Shiny app for more specific peak identification.

## When to use

Use when MALDI imaging proteolytic peaks (m/z [M+H]+ list) need to be annotated against LC-MS/MS proteomic discovery data, and a gene-name cluster list must first be used to restrict candidate biomarkers before correlation matching.

## When NOT to use

- The input is already an annotated MALDI peak-to-protein mapping table and no gene-name restriction is needed
- No LC-MS/MS-based proteomic biomarker discovery data (e.g., MaxQuant peptides.txt) is available to correlate against MALDI m/z peaks
- The analysis goal is de novo peptide or protein identification from MALDI spectra rather than correlating measured m/z peaks against discovery-data candidates

## Inputs

- peptides.txt file exported from MaxQuant (LC-MS/MS-based proteomic biomarker discovery data)
- Gene-name list of interest (e.g., HSIL(+) cluster gene names, Supplementary data 2)
- MALDI m/z [M+H]+ peak list (Supplementary data 3)

## Outputs

- Results table with 13 columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, mass difference) exported as xlsx
- Annotated MALDI imaging peak assignments restricted to the filtered gene-name set

## How to apply

Load the MaxQuant-exported peptides.txt file (LC-MS/MS proteomic biomarker discovery data), the gene-name list of interest (e.g., the HSIL(+) cluster gene names), and the MALDI m/z [M+H]+ peak list into R. Restrict the LC-MS/MS data to the supplied gene names, then run the MALDImID correlation routine (via the R package or the Shiny app) to match each MALDI m/z [M+H]+ peak against the filtered biomarker set. Assemble the results table with the 13 documented columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, and mass difference) and export as xlsx. The rationale is that filtering by the cluster gene names increases specificity of MALDI peak identification relative to matching against the full proteome.

## Related tools

- **MALDImID** (R package whose correlation routine matches MALDI m/z [M+H]+ peaks against gene-name-filtered LC-MS/MS proteomic biomarker discovery data) — https://biosciences.shinyapps.io/Maldimid
- **Shiny** (Web-app framework hosting MALDImIDshiny so the same correlation analysis can be run interactively without local R installation) — https://biosciences.shinyapps.io/Maldimid
- **R** (Environment in which the MALDImID package is executed to filter and correlate the LC-MS/MS data with MALDI peak lists) — budczies/MALDImIDshiny

## Evaluation signals

- The produced xlsx table contains the 13 documented columns including MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, MALDI mass, and mass difference
- Row-by-row comparison against the deposited reference result (Supplementary data 4) matches on MALDI m/z [M+H]+ values, gene names, and mass difference columns
- Every gene name in the output table belongs to the supplied gene-name list of interest (e.g., the HSIL(+) cluster), confirming the filter was applied
- Each MALDI output row retains a computable mass difference between the MALDI mass and the LC-MS mass

## Limitations

- The package is made available under the GPL 3 license, which constrains redistribution of derivative code
- Identification specificity depends on the quality and completeness of the gene-name list; peaks without a matching filtered biomarker candidate remain unassigned
- The repository README (budczies/MALDImIDshiny) is essentially empty, so local deployment guidance must come from the article's supplementary workflow

## Evidence

- [abstract] abstract: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid"
- [supplementary] supplementary: [evidence span withheld — anchor status: not_found]
