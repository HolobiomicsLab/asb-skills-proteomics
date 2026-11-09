---
name: maldi-peak-protein-correlation
description: Use when proteolytic peaks from bottom-up MALDI imaging must be assigned
  more specific protein identities by correlating MALDI m/z [M+H]+ peaks with LC-MS/MS
  biomarker discovery data restricted to a gene-name list of interest.
when_to_use_negative:
- Input MALDI m/z peaks cannot be paired with any LC-MS/MS-based proteomic biomarker
  discovery data, since the correlation routine requires that counterpart data
- The goal is de novo MALDI peak annotation without a restricted gene-name list of
  interest, which is outside the correlated-match scope demonstrated here
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MALDImID
  role: R package implementing the correlation routine that matches MALDI m/z [M+H]+
    peaks against LC-MS/MS proteomic biomarker discovery data
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Environment in which the MALDImID correlation routine is executed on the three
    supplied inputs
- name: Shiny
  role: Web interface exposing the MALDImID correlation workflow directly as the Maldimid
    app
  repo: https://biosciences.shinyapps.io/Maldimid
- name: MALDImIDshiny
  role: GitHub repository accompanying the MALDImID Shiny app
  repo: budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:89b6bc83f8223056c1698a3d51f42f3b4f8ace92a39e08799da86e6b6dba7786
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - MALDImIDshiny
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# maldi-peak-protein-correlation

## Summary

MALDImID correlation-based identification of proteolytic peaks in bottom-up MALDI imaging data by matching MALDI m/z [M+H]+ values against LC-MS/MS-based proteomic biomarker discovery data, provided as an R package and Shiny app.

## When to use

Use when proteolytic peaks from bottom-up MALDI imaging must be assigned more specific protein identities by correlating MALDI m/z [M+H]+ peaks with LC-MS/MS biomarker discovery data restricted to a gene-name list of interest.

## When NOT to use

- Input MALDI m/z peaks cannot be paired with any LC-MS/MS-based proteomic biomarker discovery data, since the correlation routine requires that counterpart data
- The goal is de novo MALDI peak annotation without a restricted gene-name list of interest, which is outside the correlated-match scope demonstrated here

## Inputs

- peptides.txt file exported from MaxQuant (LC-MS/MS proteomic biomarker discovery data, Supplementary data 1)
- Gene-name list of interest, e.g. the HSIL(+) cluster gene names (Supplementary data 2)
- List of MALDI m/z [M+H]+ peak values from MALDI imaging data (Supplementary data 3)

## Outputs

- Annotated correlation results table (xlsx) with 13 columns: MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, and mass difference
- Reference MALDImID result table for validation (Supplementary data 4)

## How to apply

Load the three supplied inputs into R: the peptides.txt file exported from MaxQuant (Supplementary data 1), the gene-name list of the cluster of interest such as the HSIL(+) cluster (Supplementary data 2), and the MALDI m/z list (Supplementary data 3). Execute the MALDImID R package correlation routine to match each MALDI m/z [M+H]+ peak against the LC-MS/MS-based proteomic biomarker discovery data, restricted to the cluster's gene names; the same routine is exposed through the MALDImID Shiny app. Assemble the results table with the 13 documented columns: MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, and mass difference. Export the annotated table as an xlsx file. Validate by comparing the produced xlsx row-by-row against the deposited reference result (Supplementary data 4), checking that MALDI m/z [M+H]+ values, gene names, and mass difference columns match the reference deposit.

## Related tools

- **MALDImID** (R package implementing the correlation routine that matches MALDI m/z [M+H]+ peaks against LC-MS/MS proteomic biomarker discovery data) — https://biosciences.shinyapps.io/Maldimid
- **R** (Environment in which the MALDImID correlation routine is executed on the three supplied inputs)
- **Shiny** (Web interface exposing the MALDImID correlation workflow directly as the Maldimid app) — https://biosciences.shinyapps.io/Maldimid
- **MALDImIDshiny** (GitHub repository accompanying the MALDImID Shiny app) — budczies/MALDImIDshiny

## Evaluation signals

- The produced xlsx table matches the deposited reference MALDImID table (Supplementary data 4) row-by-row
- MALDI m/z [M+H]+ values in the output match those in the input MALDI m/z list
- Gene names in the output are restricted to the supplied cluster gene-name list (e.g. HSIL(+) cluster)
- The output contains the 13 documented columns including LC-MS score, unique peptide, per-condition averages, positive replicate counts, MALDI mass, and mass difference
- Mass difference column values equal the difference between MALDI mass and LC-MS mass for each matched row

## Limitations

- Identification depends on the availability and quality of LC-MS/MS-based proteomic biomarker discovery data as the correlation counterpart
- The correlation is restricted to a supplied gene-name list of interest, so peaks outside that list will not be annotated
- Validation relies on comparison against a deposited reference result; deviations in any of the three inputs (peptides.txt, gene-name list, m/z list) will change the output

## Evidence

- [abstract] R package and Shiny app purpose: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] Shiny app availability: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [abstract] License and availability: "The package is made available under the GPL 3 license."
