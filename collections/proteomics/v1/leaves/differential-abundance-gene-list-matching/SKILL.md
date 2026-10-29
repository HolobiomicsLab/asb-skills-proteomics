---
name: differential-abundance-gene-list-matching
description: Use when you have a mass-matched peptide identification table and a list
  of gene names from an upstream differential-abundance step, and need to keep only
  peptides attributable to proteins of the tissue-of-interest signature (e.g., HSIL(+)
  cluster).
when_to_use_negative:
- Input is already filtered to the tissue-of-interest protein list, so a second gene-name
  match would be redundant
- You lack a differential-abundance signature gene list and only have raw peak lists
  or unannotated spectra — the gene-name matching step cannot be performed
- Your peptide table has no 'Gene names' column (e.g., protein-level or unannotated
  output), since matching relies on that field
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MALDImID
  role: R package implementing the LC-MS/MS-based proteomic biomarker discovery workflow
    that includes the gene-name-based filtering of mass-matched peptide identifications
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language used to load the peptides.txt table, match 'Gene names' against the
    signature gene list (splitting multi-gene ';' entries), subset rows, and write
    the filtered CSV
- name: Shiny
  role: Web interface through which the MALDImID workflow can be executed without
    local installation
  repo: https://biosciences.shinyapps.io/Maldimid
- name: MALDImIDshiny
  role: GitHub repository for the Shiny app implementation of the MALDImID workflow
  repo: https://github.com/budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:151f38e14c35532f55d50a3225d4979a0ff1f0d9fd4d432083d507a8c6ea1d36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - MALDImIDshiny
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# differential-abundance-gene-list-matching

## Summary

Restricts mass-matched peptide identifications (e.g., a MaxQuant-style peptides.txt table) to proteins significantly more abundant in a tissue of interest by matching the 'Gene names' field against a differential-abundance signature gene list, as implemented in the MALDImID R workflow for MALDI imaging peak identification.

## When to use

Use when you have a mass-matched peptide identification table and a list of gene names from an upstream differential-abundance step, and need to keep only peptides attributable to proteins of the tissue-of-interest signature (e.g., HSIL(+) cluster).

## When NOT to use

- Input is already filtered to the tissue-of-interest protein list, so a second gene-name match would be redundant
- You lack a differential-abundance signature gene list and only have raw peak lists or unannotated spectra — the gene-name matching step cannot be performed
- Your peptide table has no 'Gene names' column (e.g., protein-level or unannotated output), since matching relies on that field

## Inputs

- Mass-matched peptide identification table (MaxQuant-style peptides.txt) loaded as an R data frame
- Gene-name list of proteins significantly more abundant in the tissue of interest (e.g., HSIL(+) cluster signature from the upstream differential-abundance step)

## Outputs

- Filtered peptide-identification table (hsil_positive_peptides.csv) containing only peptides whose 'Gene names' field contains at least one signature gene name

## How to apply

Load the mass-matched peptide table (MaxQuant-style peptides.txt) into R as a data frame and load the signature gene-name list produced by the upstream differential-abundance step (e.g., proteins significantly more abundant in the HSIL(+) tissue cluster). Split each row's 'Gene names' field on the standard ';' separator used for multi-gene entries, then retain rows with at least one gene-name hit against the signature list. Subset the peptide table to the matched rows using base R or dplyr filtering and write the result to hsil_positive_peptides.csv. The rationale, from the MALDImID approach, is that linking LC-MS/MS proteomic biomarker discovery data to MALDI imaging peaks becomes more specific when identifications are restricted to proteins of the tissue-of-interest signature.

## Related tools

- **MALDImID** (R package implementing the LC-MS/MS-based proteomic biomarker discovery workflow that includes the gene-name-based filtering of mass-matched peptide identifications) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language used to load the peptides.txt table, match 'Gene names' against the signature gene list (splitting multi-gene ';' entries), subset rows, and write the filtered CSV)
- **Shiny** (Web interface through which the MALDImID workflow can be executed without local installation) — https://biosciences.shinyapps.io/Maldimid
- **MALDImIDshiny** (GitHub repository for the Shiny app implementation of the MALDImID workflow) — https://github.com/budczies/MALDImIDshiny

## Examples

```
hsil <- peptides[grepl(paste(sig_genes, collapse="|"), peptides[["Gene names"]]),]; write.csv(hsil, "hsil_positive_peptides.csv", row.names=FALSE)
```

## Evaluation signals

- Output file hsil_positive_peptides.csv exists after the run
- Every retained row's 'Gene names' field contains at least one gene name from the HSIL(+) signature list (invariant: no row without a match survives)
- Row count of the filtered table is less than or equal to the input mass-matched table, with matched rows preserved unchanged
- Multi-gene entries are correctly handled: rows matching a signature gene after splitting on ';' are retained even when other listed genes are not in the signature
- Filtered output columns and formatting are consistent with the input peptides.txt schema

## Limitations

- The filter is only as specific as the upstream differential-abundance signature: peptides are retained if any listed gene matches, so shared/multi-gene entries can pull in peptides partially attributable to non-signature proteins
- Requires a 'Gene names' field with the standard ';' multi-gene separator (MaxQuant-style); tables lacking this annotation cannot be filtered this way
- The README content for budczies/MALDImIDshiny provided here is essentially empty, so repository-level usage guidance beyond the article is unavailable

## Evidence

- [abstract] MALDImID implements gene-name filtering of mass-matched peptide identifications: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
- [abstract] MALDImID app availability: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid"
- [abstract] License of the MALDImID package: "The package is made available under the GPL 3 license."
