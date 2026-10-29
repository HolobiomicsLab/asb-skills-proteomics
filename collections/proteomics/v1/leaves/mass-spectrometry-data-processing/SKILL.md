---
name: mass-spectrometry-data-processing
description: Use when a mass-matched peptide identification table must be restricted
  to proteins from a differential-abundance signature list before annotating MALDI
  imaging peaks with LC-MS/MS-derived identifications.
when_to_use_negative:
- The peptide table is already restricted to the proteins of interest and needs no
  gene-name-based subsetting
- The input is raw MALDI imaging or LC-MS/MS spectral data rather than an annotated,
  mass-matched peptide identification table
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MALDImID
  role: R package implementing the filtering of mass-matched peptide identifications
    to proteins of interest for MALDI imaging peak identification
- name: MALDImIDshiny
  role: Shiny app providing the same gene-name-based peptide filtering workflow interactively
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language in which the peptide table is loaded, matched against the gene-name
    list, subsetted (base R or dplyr), and written to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:ddcb1a59ee26d3e2a2af168408e88b6209aabb5fe99d4f8f65f8ae4592918703
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - MALDImIDshiny
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mass-spectrometry-data-processing

## Summary

Filter mass-matched peptide identifications (e.g., a MaxQuant-style peptides.txt table) down to peptides whose 'Gene names' field contains at least one gene from a supplied signature protein list, as done in the MALDImID workflow to restrict MALDI imaging peak candidates to proteins enriched in a tissue of interest (e.g., the HSIL(+) cluster).

## When to use

Use when a mass-matched peptide identification table must be restricted to proteins from a differential-abundance signature list before annotating MALDI imaging peaks with LC-MS/MS-derived identifications.

## When NOT to use

- The peptide table is already restricted to the proteins of interest and needs no gene-name-based subsetting
- The input is raw MALDI imaging or LC-MS/MS spectral data rather than an annotated, mass-matched peptide identification table

## Inputs

- Mass-matched peptide identification table (MaxQuant-style peptides.txt) loaded as an R data frame
- Gene-name list of proteins significantly more abundant in the tissue of interest (e.g., HSIL(+) cluster signature gene names)

## Outputs

- Filtered peptide-identification table restricted to peptides from the supplied signature proteins (e.g., hsil_positive_peptides.csv)

## How to apply

Load the mass-matched peptide identification table (a MaxQuant-style peptides.txt output) into R as a data frame, and load the gene-name list of proteins significantly more abundant in the tissue of interest (the signature list from the upstream differential-abundance step, e.g., HSIL(+) cluster gene names). Split each peptide row's 'Gene names' field on the standard ';' separator and match against the signature list, retaining rows with at least one gene-name hit. Subset the peptide table to the matched rows using base R or dplyr filtering, then write the filtered table (e.g., to hsil_positive_peptides.csv). This gene-name-based restriction is the MALDImID strategy for more specific identification of MALDI imaging proteolytic peaks using LC-MS/MS-based proteomic biomarker discovery data; the MALDImID package and Shiny app implement this exploitation of biomarker discovery data.

## Related tools

- **MALDImID** (R package implementing the filtering of mass-matched peptide identifications to proteins of interest for MALDI imaging peak identification)
- **MALDImIDshiny** (Shiny app providing the same gene-name-based peptide filtering workflow interactively) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language in which the peptide table is loaded, matched against the gene-name list, subsetted (base R or dplyr), and written to CSV)

## Examples

```
hsil <- peptides[grepl(paste(genes_of_interest, collapse="|"), peptides$`Gene names`), ]; write.csv(hsil, "hsil_positive_peptides.csv")
```

## Evaluation signals

- The output file (e.g., hsil_positive_peptides.csv) exists and is non-empty
- Every retained row's 'Gene names' field contains at least one gene name from the signature list after splitting multi-gene entries on ';'
- No row in the output carries a 'Gene names' field whose entries all fall outside the supplied signature gene-name list
- Row count of the output is less than or equal to the input peptides.txt row count, with the reduction traceable to the gene-name filter
- Column schema of the output matches the input peptides.txt table (filtering removes rows, not columns)

## Limitations

- Filtering depends entirely on the quality of the supplied gene-name signature list; peptides from proteins absent from the list are discarded even if mass-matched
- Multi-gene entries must be handled correctly (split on ';'); naive string matching can mis-retain or mis-exclude peptides assigned to multiple proteins
- The peptide input must already be mass-matched against the biomarker discovery data; this skill does not perform the mass matching itself

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid.: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [abstract] The package is made available under the GPL 3 license.: "The package is made available under the GPL 3 license."
