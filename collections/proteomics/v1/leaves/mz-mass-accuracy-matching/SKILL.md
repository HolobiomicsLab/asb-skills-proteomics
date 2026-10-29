---
name: mz-mass-accuracy-matching
description: Use when a list of MALDI m/z [M+H]+ peaks from MALDI imaging must be
  annotated with peptide/protein identities by correlating against LC-MS/MS proteomic
  discovery data (e.g. MaxQuant peptides.txt) restricted to a candidate gene-name
  list.
when_to_use_negative:
- When no LC-MS/MS-based proteomic biomarker discovery data (e.g. MaxQuant peptides.txt)
  are available to correlate against the MALDI m/z peaks.
- When the goal is generic MALDI peak picking or de novo identification rather than
  matching peaks to candidate peptides/proteins from a predefined gene-name list.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MALDImID
  role: R package implementing the correlation routine that matches MALDI m/z [M+H]+
    peaks against LC-MS/MS proteomic discovery data
  repo: https://biosciences.shinyapps.io/Maldimid
- name: Shiny
  role: Web app front-end (MALDImIDshiny) for running the same peak-identification
    correlation interactively
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Runtime environment in which the MALDImID correlation analysis is executed
    and the results xlsx is assembled
  repo: budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:1bef67956aecc03b17b05ff2353246fc02ca9cfa65849a2e6647981e477a1858
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mz-mass-accuracy-matching

## Summary

Mass-accuracy matching of MALDI imaging proteolytic [M+H]+ peaks against LC-MS/MS-based proteomic biomarker discovery data using the MALDImID R package and Shiny app, to achieve more specific identification of MALDI imaging peaks.

## When to use

Use when a list of MALDI m/z [M+H]+ peaks from MALDI imaging must be annotated with peptide/protein identities by correlating against LC-MS/MS proteomic discovery data (e.g. MaxQuant peptides.txt) restricted to a candidate gene-name list.

## When NOT to use

- When no LC-MS/MS-based proteomic biomarker discovery data (e.g. MaxQuant peptides.txt) are available to correlate against the MALDI m/z peaks.
- When the goal is generic MALDI peak picking or de novo identification rather than matching peaks to candidate peptides/proteins from a predefined gene-name list.

## Inputs

- peptides.txt file exported from MaxQuant (LC-MS/MS proteomic biomarker discovery data; Supplementary data 1)
- Gene-name list of interest (e.g. HSIL(+) cluster gene names; Supplementary data 2)
- MALDI m/z peak list ([M+H]+ values; Supplementary data 3)

## Outputs

- Results table with 13 columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, mass difference), exported as xlsx (reference deposit: Supplementary data 4)

## How to apply

Load three inputs into R: the peptides.txt file exported from MaxQuant (Supplementary data 1), the HSIL(+) cluster gene-name list of interest (Supplementary data 2), and the MALDI m/z list (Supplementary data 3). Run the MALDImID correlation routine, which matches each MALDI m/z [M+H]+ peak against the LC-MS/MS-based proteomic biomarker discovery data while restricting the search to the supplied gene names — the gene-list restriction is the step that gives 'more specific identification' of bottom-up MALDI imaging peaks. Assemble the results table with the 13 documented columns: MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, and mass difference. Export the annotated table as an xlsx file. The same routine is available interactively through the Shiny app at https://biosciences.shinyapps.io/Maldimid; the package itself is distributed under the GPL 3 license.

## Related tools

- **MALDImID** (R package implementing the correlation routine that matches MALDI m/z [M+H]+ peaks against LC-MS/MS proteomic discovery data) — https://biosciences.shinyapps.io/Maldimid
- **Shiny** (Web app front-end (MALDImIDshiny) for running the same peak-identification correlation interactively) — https://biosciences.shinyapps.io/Maldimid
- **R** (Runtime environment in which the MALDImID correlation analysis is executed and the results xlsx is assembled) — budczies/MALDImIDshiny

## Evaluation signals

- Row-by-row comparison of the produced xlsx table against the deposited reference result (Supplementary data 4): MALDI m/z [M+H]+ values, gene names, and mass difference columns must match the reference deposit.
- Output table contains exactly the 13 documented columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, mass difference).
- Every matched row must carry a gene name drawn from the supplied candidate gene-name list (e.g. the HSIL(+) cluster).
- Mass difference column is populated for matched peaks, reflecting agreement between MALDI mass and LC-MS mass.
- Shiny-app and R-package runs on the same three inputs produce comparable result tables.

## Limitations

- The gene-name list restriction means peaks can only be assigned to candidates present in the supplied list; matches outside it are not reported.
- Identifications are only as specific as the underlying LC-MS/MS proteomic biomarker discovery data (MaxQuant peptides.txt); no LC-MS/MS discovery data, no matching.
- The repository README for budczies/MALDImIDshiny contains no additional usage documentation, limiting locally reproducible setup guidance beyond the hosted Shiny app.

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] Shiny app availability: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [abstract] License: "The package is made available under the GPL 3 license."
- [supplementary] Workflow from source card task_001: [evidence span withheld — anchor status: not_found]
