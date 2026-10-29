---
name: maxquant-peptide-table-parsing
description: Use when a MaxQuant-exported peptides.txt table must be loaded into R
  and paired with a MALDI m/z [M+H]+ peak list and a gene-name list to identify MALDI
  imaging proteolytic peaks.
when_to_use_negative:
- When no MaxQuant peptides.txt export of LC-MS/MS biomarker discovery data is available
  — MALDImID specifically exploits LC-MS/MS-based data and cannot identify peaks from
  MALDI m/z alone.
- When the input is already a finished MALDImID results table (e.g. Supplementary
  data 4) rather than raw peptide/peak inputs — no parsing or correlation is needed.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MALDImID
  role: R package whose correlation routine matches MALDI m/z [M+H]+ peaks against
    the MaxQuant-derived LC-MS/MS peptide data restricted to gene names of interest
- name: R
  role: Environment in which the peptides.txt input is loaded and the MALDImID correlation
    routine is executed
- name: Shiny
  role: Web interface (MALDImIDshiny) offering the same MALDImID analysis without
    local installation
  repo: https://biosciences.shinyapps.io/Maldimid
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:643903c2b815b4643102a01e600614a248570fddb037e136a538cc567278ef96
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MALDImID
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# maxquant-peptide-table-parsing

## Summary

Parse and use the peptides.txt table exported from MaxQuant as the LC-MS/MS-based proteomic biomarker discovery input to MALDImID, so that MALDI imaging m/z [M+H]+ peaks can be matched to peptide masses restricted to gene names of interest.

## When to use

Use when a MaxQuant-exported peptides.txt table must be loaded into R and paired with a MALDI m/z [M+H]+ peak list and a gene-name list to identify MALDI imaging proteolytic peaks.

## When NOT to use

- When no MaxQuant peptides.txt export of LC-MS/MS biomarker discovery data is available — MALDImID specifically exploits LC-MS/MS-based data and cannot identify peaks from MALDI m/z alone.
- When the input is already a finished MALDImID results table (e.g. Supplementary data 4) rather than raw peptide/peak inputs — no parsing or correlation is needed.

## Inputs

- peptides.txt table exported from MaxQuant (LC-MS/MS proteomic biomarker discovery data)
- gene-name list of interest (e.g. HSIL(+) cluster gene names)
- MALDI m/z [M+H]+ peak list from MALDI imaging data

## Outputs

- MALDImID results table with 13 columns (MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages, positive replicate counts, MALDI mass, mass difference)
- Annotated xlsx file of the results table

## How to apply

Load the peptides.txt file exported from MaxQuant (Supplementary data 1) into R alongside the gene-name list of interest (e.g. the HSIL(+) cluster gene names) and the MALDI m/z [M+H]+ peak list. Run the MALDImID R package correlation routine, which matches each MALDI m/z [M+H]+ peak against the LC-MS/MS-based proteomic biomarker discovery data restricted to the supplied gene names. Assemble the results table with the 13 documented columns: MALDI m/z [M+H]+, LC-MS mass, gene names, LC-MS score, unique peptide, per-condition averages and positive replicate counts, MALDI mass, and mass difference. Export the annotated table as an xlsx file. The rationale is that LC-MS/MS peptide masses from MaxQuant provide more specific identification of bottom-up MALDI imaging peaks than MALDI mass alone; the Shiny app (MALDImIDshiny) exposes the same analysis interactively.

## Related tools

- **MALDImID** (R package whose correlation routine matches MALDI m/z [M+H]+ peaks against the MaxQuant-derived LC-MS/MS peptide data restricted to gene names of interest)
- **R** (Environment in which the peptides.txt input is loaded and the MALDImID correlation routine is executed)
- **Shiny** (Web interface (MALDImIDshiny) offering the same MALDImID analysis without local installation) — https://biosciences.shinyapps.io/Maldimid

## Examples

```
peptides <- read.delim("peptides.txt")
```

## Evaluation signals

- Row-by-row comparison of the exported xlsx against the deposited reference MALDImID table (Supplementary data 4), verifying MALDI m/z [M+H]+, gene names, and mass difference columns match.
- Output table contains all 13 documented columns in the expected schema.
- Every matched row carries a gene name present in the supplied gene-name list (e.g. HSIL(+) cluster genes).
- Mass difference values in the output are consistent with MALDI mass vs LC-MS mass per matched peptide.

## Limitations

- Identification specificity depends on restricting the LC-MS/MS data to the gene-name list; an incomplete or off-target gene list changes which MALDI peaks can be matched.
- The package is available under the GPL 3 license, which may constrain redistribution in some pipelines.

## Evidence

- [abstract] abstract: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data"
