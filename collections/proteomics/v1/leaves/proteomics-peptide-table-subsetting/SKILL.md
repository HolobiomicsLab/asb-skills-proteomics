---
name: proteomics-peptide-table-subsetting
description: Use when you have a mass-matched peptide identification table (e.g. a
  MaxQuant-style peptides.txt) and need to keep only peptides whose 'Gene names' field
  matches a tissue-signature protein list before MALDI imaging peak annotation.
when_to_use_negative:
- When no tissue-of-interest protein/gene-name list is available — filtering would
  empty or arbitrarily truncate the peptide table.
- When the input is already a filtered/annotated feature table containing only peptides
  of interest — re-subsetting adds no information.
- When the peptide table lacks a 'Gene names' field (e.g. different proteomics search-engine
  output), since the matching step depends on that column.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MALDImID
  role: R package implementing the proteomic biomarker discovery workflow whose filtering
    step restricts mass-matched peptide identifications to proteins of interest for
    MALDI imaging peak identification
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Implementation language used to load peptides.txt as a data frame, split multi-gene
    entries on ';', and subset matched rows
- name: Shiny
  role: Web front end exposing the MALDImID workflow so the filtering can be applied
    interactively
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:8dfde5aabe2807317e627ba441775ab09d040ae6a671e11afb18798ac6a11beb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# proteomics-peptide-table-subsetting

## Summary

A gene-name-based filtering step from the MALDImID workflow that restricts a mass-matched MaxQuant-style 'peptides.txt' identification table to peptides belonging to a supplied list of proteins (gene names) of interest, so that MALDI imaging peak assignments are specific to the tissue-of-interest signature.

## When to use

Use when you have a mass-matched peptide identification table (e.g. a MaxQuant-style peptides.txt) and need to keep only peptides whose 'Gene names' field matches a tissue-signature protein list before MALDI imaging peak annotation.

## When NOT to use

- When no tissue-of-interest protein/gene-name list is available — filtering would empty or arbitrarily truncate the peptide table.
- When the input is already a filtered/annotated feature table containing only peptides of interest — re-subsetting adds no information.
- When the peptide table lacks a 'Gene names' field (e.g. different proteomics search-engine output), since the matching step depends on that column.

## Inputs

- Mass-matched peptide identification table in MaxQuant 'peptides.txt' format with a 'Gene names' column
- Gene-name list of proteins significantly more abundant in the tissue of interest (e.g. HSIL(+) cluster signature gene names)

## Outputs

- Filtered peptide-identification table restricted to peptides from proteins of interest (e.g. hsil_positive_peptides.csv)

## How to apply

Load the mass-matched peptide identification table (MaxQuant-style 'peptides.txt') into R as a data frame, and load the gene-name list of proteins significantly more abundant in the tissue of interest (e.g. the HSIL(+) cluster signature produced by the upstream differential-abundance step). Split each peptide row's 'Gene names' field on the standard ';' separator to handle multi-gene entries, and retain rows with at least one gene-name hit against the signature list. Subset the table to matched rows using base R or dplyr, then write the result (e.g. hsil_positive_peptides.csv). The rationale, per the MALDImID approach, is to make identification of MALDI imaging proteolytic peaks more specific by tying candidate masses to LC-MS/MS-based proteomic biomarker discovery data restricted to the proteins of interest.

## Related tools

- **MALDImID** (R package implementing the proteomic biomarker discovery workflow whose filtering step restricts mass-matched peptide identifications to proteins of interest for MALDI imaging peak identification) — https://biosciences.shinyapps.io/Maldimid
- **R** (Implementation language used to load peptides.txt as a data frame, split multi-gene entries on ';', and subset matched rows)
- **Shiny** (Web front end exposing the MALDImID workflow so the filtering can be applied interactively) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
hsil <- read.csv('hsil_genes.txt')$gene; pep <- read.delim('peptides.txt'); keep <- sapply(strsplit(pep[['Gene.names']], ';'), function(g) any(g %in% hsil)); write.csv(pep[keep, ], 'hsil_positive_peptides.csv')
```

## Evaluation signals

- Every retained row's 'Gene names' field contains at least one gene name from the tissue-of-interest signature list (after splitting on ';').
- No row in the output table has a 'Gene names' field entirely disjoint from the supplied gene-name list.
- The output file (e.g. hsil_positive_peptides.csv) exists and is a valid CSV with the same columns as the input peptides.txt.
- Row count of the output is less than or equal to the input table, and the set difference contains only rows whose gene names are absent from the signature list.
- Multi-gene entries (semicolon-separated) were matched rather than dropped, i.e. peptides annotated to both a signature protein and another protein are retained.

## Limitations

- Peptides annotated only with gene names outside the supplied signature list are discarded, so sensitivity depends entirely on the completeness of the upstream differential-abundance gene list.
- The filter relies on the MaxQuant-style 'Gene names' column and ';' separator; tables from other search engines may need column mapping before the step works.
- The repository README for budczies/MALDImIDshiny contains no additional documentation, limiting implementation detail beyond the article text.

## Evidence

- [abstract] R package and Shiny app for LC-MS/MS-based biomarker data exploitation: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] Purpose of more specific MALDI imaging peak identification: [evidence span withheld — anchor status: not_found]
- [abstract] Shiny app availability for interactive use: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
