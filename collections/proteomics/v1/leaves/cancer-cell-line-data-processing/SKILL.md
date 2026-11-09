---
name: cancer-cell-line-data-processing
description: Use when a pancancer expression matrix from cell line models (e.g., DepMap/GDSC/NCI60)
  must be joined to cancer-type metadata and summarized per cancer type for a specific
  gene such as TP53.
when_to_use_negative:
- The analysis goal is drug-omics correlation or drug response prediction rather than
  per-cancer-type expression summary — use IMOPAC's drug-omics correlation or drug
  response prediction modules instead
- The data are patient tumour cohorts rather than patient-derived cancer cell line
  models (IMOPAC covers NCI-60, GDSC and DepMap cell lines; PDX/organoid datasets
  are not yet integrated)
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: IMOPAC
  role: Web server providing the 'Cancer Types Summary' pancancer expression module
    with boxplot visualization of a gene's expression across cancer types
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: R
  role: Backend language for loading expression RData, extracting the gene row, joining
    metadata, computing grouped median/mean statistics with dplyr, and rendering SVG
    figures
- name: dplyr
  role: R package for grouping TP53 expression by cancer type and computing median/mean
    summary statistics per cancer type
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:6da659496a10f025d791f9d16ddf9dfc4d558bdde9ce98393700841e26825a4c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wolfgangsk07/IMOPAC
  repo_url: https://github.com/wolfgangsk07/IMOPAC
  tools:
  - IMOPAC
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cancer-cell-line-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Workflow for processing pancancer cancer cell line multiomics matrices (e.g., DepMap mRNA expression across 1773 cell lines from 34 cancer types) to summarize and visualize gene-level abundance per cancer type, as implemented in the IMOPAC 'Cancer Types Summary' pancancer expression module.

## When to use

Use when a pancancer expression matrix from cell line models (e.g., DepMap/GDSC/NCI60) must be joined to cancer-type metadata and summarized per cancer type for a specific gene such as TP53.

## When NOT to use

- The analysis goal is drug-omics correlation or drug response prediction rather than per-cancer-type expression summary — use IMOPAC's drug-omics correlation or drug response prediction modules instead
- The data are patient tumour cohorts rather than patient-derived cancer cell line models (IMOPAC covers NCI-60, GDSC and DepMap cell lines; PDX/organoid datasets are not yet integrated)

## Inputs

- Pancancer cancer cell line mRNA expression matrix (e.g., DepMap transcriptomic data covering 1773 cell lines across 34 cancer types, distributed as .RData)
- Cell line metadata mapping each cell line to its cancer type
- Target gene identifier (e.g., TP53)

## Outputs

- Ranked per-cancer-type summary table (median/mean expression), e.g., result_tp53_mrna_pancancer.csv
- Boxplot of expression abundance across cancer types, e.g., tp53_mrna_pancancer_boxplot.svg

## How to apply

Load the pancancer mRNA expression matrix (DepMap: 12 omics, 1773 cell lines, 34 cancer types) in R using dplyr/stringr/tidyr from the IMOPAC-processed RData resources. Extract the target gene row (e.g., TP53) and join it with cell line metadata mapping cell lines to cancer types. Group expression by cancer type and compute median/mean summary statistics, then generate a boxplot of expression abundance across all cancer types as done in the IMOPAC 'Cancer Types Summary' module. Rank cancer types by median expression in descending order and export the ranked table and boxplot (e.g., CSV and SVG). Validation is by confirming top-ranked cancer types match known results — for TP53, highest mRNA in lymphoma, mesothelioma, and lymphoblastic leukaemia in the GDSC database.

## Related tools

- **IMOPAC** (Web server providing the 'Cancer Types Summary' pancancer expression module with boxplot visualization of a gene's expression across cancer types) — https://github.com/wolfgangsk07/IMOPAC
- **R** (Backend language for loading expression RData, extracting the gene row, joining metadata, computing grouped median/mean statistics with dplyr, and rendering SVG figures)
- **dplyr** (R package for grouping TP53 expression by cancer type and computing median/mean summary statistics per cancer type)

## Examples

```
Rscript -e 'library(dplyr); tp53 <- expr["TP53",]; bytype <- inner_join(tp53, meta, by="cell_line"); ranked <- bytype %>% group_by(cancer_type) %>% summarise(median=median(expr)); write.csv(ranked,"result_tp53_mrna_pancancer.csv")'
```

## Evaluation signals

- Top-ranked cancer types by median TP53 mRNA level correspond to lymphoma, mesothelioma, and lymphoblastic leukaemia (as reported for the GDSC database)
- Ranked summary table (e.g., result_tp53_mrna_pancancer.csv) contains one row per cancer type with median and mean expression values
- Boxplot output is a valid SVG (figures on the IMOPAC website are generated by SVG.js v2.7) covering all queried cancer types
- Number of cancer types matches the source database coverage (34 for DepMap, 28 for GDSC, 9 for NCI60)

## Limitations

- Dataset files (*.RData) are not included in the GitHub repository due to file size limits; users must download them via urls.txt
- Coverage differs per source database: DepMap has 22 drugs, GDSC 173 drugs, NCI60 695 drugs — pancancer summaries are bounded by each dataset's cancer-type panel
- Planned future datasets (PDX and organoid multiomics and drug screens) are not yet integrated, so results are specific to patient-derived cell line models

## Evidence

- [methods] Pancancer expression boxplot workflow: [evidence span withheld — anchor status: not_found]
- [results] TP53 pancancer result: "the mRNA level of TP53 is the highest in lymphoma, mesothelioma, and lymphoblastic leukaemia in the GDSC database (Fig. 2B)"
- [methods] DepMap dataset coverage: "12 omics, 1773 cell lines from 34 cancer types and 22 drugs are available in DepMap database"
- [abstract] IMOPAC analysis scope: "The analyses in our portal include pancancer expression, drug-omics/pathway correlation, cancer subtypes, omics-omics (cis-/trans-regulation) correlation, fusion query analysis, and drug response"
- [readme] README R environment: "You can get the urls of all RDatas in urls.txt. It could running on any web server, like apache or IIS."
