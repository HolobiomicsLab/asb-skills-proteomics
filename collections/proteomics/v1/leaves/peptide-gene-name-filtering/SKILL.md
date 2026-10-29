---
name: peptide-gene-name-filtering
description: Use when a mass-matched peptide identification table (MaxQuant-style
  'peptides.txt') must be restricted to peptides whose 'Gene names' field contains
  at least one gene from a supplied tissue-signature protein list before downstream
  MALDI-MS peak annotation.
when_to_use_negative:
- The peptide table has already been restricted to the gene names of interest, so
  a second filtering pass would be redundant.
- The input lacks a 'Gene names' field (multi-gene entries separated by ';'), making
  the gene-name matching impossible.
- No upstream differential-abundance signature gene list is available to define the
  proteins of interest.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MALDImID
  role: R package whose workflow implements the gene-name-based filtering of mass-matched
    peptide identifications against a tissue-signature protein list
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language used to load 'peptides.txt', match 'Gene names' entries against the
    signature list, and subset the table
- name: Shiny
  role: Web interface exposing the MALDImID filtering workflow for interactive use
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:014e8b78bd25cd499ad7809b92ce9fe26d6b6349860ed669d59397db742ecc7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# peptide-gene-name-filtering

## Summary

A gene-name-based filtering step from the MALDImID R workflow that restricts mass-matched peptide identifications in a MaxQuant-style 'peptides.txt' table to proteins (gene names) significantly more abundant in the tissue of interest, so MALDI-MS peak assignment is limited to a tissue-signature protein list (e.g., HSIL(+) cluster gene names).

## When to use

Use when a mass-matched peptide identification table (MaxQuant-style 'peptides.txt') must be restricted to peptides whose 'Gene names' field contains at least one gene from a supplied tissue-signature protein list before downstream MALDI-MS peak annotation.

## When NOT to use

- The peptide table has already been restricted to the gene names of interest, so a second filtering pass would be redundant.
- The input lacks a 'Gene names' field (multi-gene entries separated by ';'), making the gene-name matching impossible.
- No upstream differential-abundance signature gene list is available to define the proteins of interest.

## Inputs

- Mass-matched peptide identification table in MaxQuant-style 'peptides.txt' format with a 'Gene names' column
- Gene-name list of proteins significantly more abundant in the tissue of interest (e.g., HSIL(+) cluster signature gene names from the upstream differential-abundance step)

## Outputs

- Filtered peptide-identification table (hsil_positive_peptides.csv) containing only rows whose 'Gene names' field includes at least one gene name from the signature list

## How to apply

Load the mass-matched peptide identification table ('peptides.txt') into R as a data frame and load the gene-name list of proteins significantly more abundant in the tissue of interest (the signature list produced by the upstream differential-abundance step, e.g. HSIL(+) cluster gene names). Split multi-gene entries in each row's 'Gene names' field on the standard ';' separator and retain rows with at least one gene-name hit against the signature list. Subset the peptide table to the matched rows using base R or dplyr filtering, then write the filtered table to hsil_positive_peptides.csv. The rationale, per MALDImID, is to exploit LC-MS/MS-based proteomic biomarker discovery data for more specific identification of proteolytic peaks in bottom-up MALDI imaging data.

## Related tools

- **MALDImID** (R package whose workflow implements the gene-name-based filtering of mass-matched peptide identifications against a tissue-signature protein list) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language used to load 'peptides.txt', match 'Gene names' entries against the signature list, and subset the table)
- **Shiny** (Web interface exposing the MALDImID filtering workflow for interactive use)

## Examples

```
hsil <- peptides[grepl(paste(genes, collapse="|"), peptides[["Gene names"]]), ]; write.csv(hsil, "hsil_positive_peptides.csv")
```

## Evaluation signals

- The output file hsil_positive_peptides.csv exists after the script runs.
- Every retained row's 'Gene names' field contains at least one gene name from the HSIL(+) signature list.
- No retained row maps exclusively to gene names absent from the supplied tissue-of-interest gene list.
- Row count of the filtered table is less than or equal to the input mass-matched table, and equals the number of rows with at least one gene-name hit when re-derived independently.
- Multi-gene entries split on the ';' separator are matched correctly (a row is retained if any of its gene names hits the list, not only the first).

## Limitations

- Filtering is only as specific as the upstream gene-name list; the output is restricted to peptides from the supplied tissue-of-interest protein list (e.g., the HSIL(+) cluster gene names).
- Rows lacking a 'Gene names' annotation, or with gene names formatted differently from the signature list, will be dropped and could represent missed identifications.

## Evidence

- [abstract] we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [other] The script performs a gene-name-based filtering step: it identifies the gene names of interest within the 'peptides.txt' table: [evidence span withheld — anchor status: not_found]
