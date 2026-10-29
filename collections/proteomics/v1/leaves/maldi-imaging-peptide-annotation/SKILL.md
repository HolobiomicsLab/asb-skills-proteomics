---
name: maldi-imaging-peptide-annotation
description: Use when a mass-matched peptide identification table (e.g. MaxQuant-style
  'peptides.txt') must be filtered to peptides whose 'Gene names' field overlaps a
  differential-abundance signature list (e.g. proteins enriched in an HSIL(+) tissue
  cluster) before MALDI peak annotation.
when_to_use_negative:
- The input peptide table has not yet been mass-matched against the candidate peptide
  list from the LC-MS/MS discovery data — filtering is a post-matching restriction
  step, not an identification step
- You need de novo peak annotation without a prior protein signature list, since this
  skill requires a supplied gene-name list to filter against
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: MALDImID
  role: R package implementing the peptide filtering/annotation workflow that restricts
    mass-matched MALDI imaging peak identifications using LC-MS/MS proteomic biomarker
    discovery data
  repo: https://biosciences.shinyapps.io/Maldimid
- name: R
  role: Language in which the MALDImID package and the gene-name-based filtering of
    the 'peptides.txt' table are implemented (base R or dplyr subsetting)
- name: Shiny
  role: Web interface providing the MALDImID workflow (MALDImIDshiny) for interactive
    use of the filtering/annotation steps
  repo: budczies/MALDImIDshiny
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300005
    title: 'MALDImID: Spatialomics R package and Shiny app for more specific identification
      of MALDI imaging proteolytic peaks using LC‐MS/MS‐based proteomic biomarker
      discovery data'
  source_content_hash: sha256:ef8c2ae570c34c73fa6e55a02692f856d8bfdda1c0c32447bddb1583b7a43c02
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# maldi-imaging-peptide-annotation

## Summary

Restricts mass-matched MALDI imaging peptide identifications to peptides derived from a supplied signature protein list, using gene-name matching on a MaxQuant-style 'peptides.txt' table within the MALDImID R package/Shiny workflow. This improves specificity of bottom-up MALDI imaging peak identification by anchoring assignments to LC-MS/MS-based proteomic biomarker discovery data.

## When to use

Use when a mass-matched peptide identification table (e.g. MaxQuant-style 'peptides.txt') must be filtered to peptides whose 'Gene names' field overlaps a differential-abundance signature list (e.g. proteins enriched in an HSIL(+) tissue cluster) before MALDI peak annotation.

## When NOT to use

- The input peptide table has not yet been mass-matched against the candidate peptide list from the LC-MS/MS discovery data — filtering is a post-matching restriction step, not an identification step
- You need de novo peak annotation without a prior protein signature list, since this skill requires a supplied gene-name list to filter against

## Inputs

- Mass-matched peptide identification table in MaxQuant-style 'peptides.txt' format, including a 'Gene names' column
- Gene-name list of proteins significantly more abundant in the tissue of interest (e.g. HSIL(+) cluster signature from upstream differential-abundance analysis)

## Outputs

- Filtered peptide-identification table restricted to peptides from the supplied tissue-of-interest protein list (e.g. hsil_positive_peptides.csv)

## How to apply

Load the mass-matched peptide identification table (a MaxQuant-style 'peptides.txt' output) into R as a data frame, and load the gene-name list of proteins significantly more abundant in the tissue of interest (e.g. the HSIL(+) cluster signature from the upstream LC-MS/MS differential-abundance step). Split each row's multi-gene 'Gene names' field on the standard ';' separator and retain rows with at least one gene-name hit against the signature list, using base R or dplyr subsetting. Write the filtered table to hsil_positive_peptides.csv. The rationale, per the MALDImID paper, is to exploit LC-MS/MS-based proteomic biomarker discovery data for more specific identification of proteolytic peaks in bottom-up MALDI imaging data, so that only peptides from tissue-of-interest proteins annotate MALDI imaging peaks.

## Related tools

- **MALDImID** (R package implementing the peptide filtering/annotation workflow that restricts mass-matched MALDI imaging peak identifications using LC-MS/MS proteomic biomarker discovery data) — https://biosciences.shinyapps.io/Maldimid
- **R** (Language in which the MALDImID package and the gene-name-based filtering of the 'peptides.txt' table are implemented (base R or dplyr subsetting))
- **Shiny** (Web interface providing the MALDImID workflow (MALDImIDshiny) for interactive use of the filtering/annotation steps) — budczies/MALDImIDshiny

## Examples

```
Rscript -e 'p <- read.delim("peptides.txt"); g <- readLines("hsil_genes.txt"); keep <- sapply(strsplit(p[["Gene names"]], ";"), function(x) any(x %in% g)); write.csv(p[keep,], "hsil_positive_peptides.csv")'
```

## Evaluation signals

- The output file (e.g. hsil_positive_peptides.csv) exists after the filtering step
- Every retained row's 'Gene names' field contains at least one gene name from the HSIL(+) signature list (verified after splitting multi-gene entries on ';')
- No retained row maps exclusively to gene names outside the supplied tissue-of-interest protein list
- Row count of the filtered table is less than or equal to the input mass-matched peptide table (filtering is a strict subset operation)

## Limitations

- The filter is only as specific as the upstream differential-abundance step: peptides are retained if ANY gene name in a multi-gene entry matches the signature list, so shared/ambiguous gene annotations can leak non-target peptides through
- Requires the input to be a MaxQuant-style 'peptides.txt' table with a 'Gene names' field; other peptide identification formats are not described as supported
- No concrete filtering thresholds or tolerance parameters are specified in the provided context beyond the gene-name match criterion

## Evidence

- [abstract] R package and Shiny app for LC-MS/MS-based biomarker exploitation: "we implemented an R package and Shiny app to exploit liquid chromatography-tandem mass spectrometry (LC-MS/MS)-based proteomic biomarker discovery data for more specific identification of peaks"
- [abstract] Shiny app availability: "The Shiny app can directly be used at the following address: https://biosciences.shinyapps.io/Maldimid."
- [abstract] License of the MALDImID package: "The package is made available under the GPL 3 license."
