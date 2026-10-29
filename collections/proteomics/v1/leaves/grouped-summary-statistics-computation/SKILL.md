---
name: grouped-summary-statistics-computation
description: Use when a continuous omics value (e.g., mRNA expression) for one gene
  must be summarized per cancer type across a cell line panel (e.g., DepMap, 1773
  lines, 34 cancer types) and ranked to identify the highest-expressing groups.
when_to_use_negative:
- The goal is a single global statistic over all cell lines rather than per-cancer-type
  grouping — grouping is essential to the ranking
- The input is drug response or other non-continuous omics better suited to IMOPAC's
  drug-omics/pathway correlation module rather than the 'Cancer Types Summary' boxplot
- The gene of interest is measured via CNV alteration frequency, where the analogous
  IMOPAC module reports alteration frequencies, not grouped expression medians
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: IMOPAC
  role: Web server whose 'Cancer Types Summary' module provides the reference pancancer
    expression boxplot of a specific input gene across multiple cancer types
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: dplyr
  role: R package (v1.0.7) used to group expression by cancer type and compute median/mean
    summary statistics
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: SVG.js
  role: Library (v2.7) behind the SVG-format figures IMOPAC produces, matching the
    boxplot output format
  repo: https://github.com/wolfgangsk07/IMOPAC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:3b9ee728c118a4782a220c5bd62526c5bcbba23127380520ec7c968999a75d75
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
  - SVG.js
  - dplyr
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# grouped-summary-statistics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-group summary statistics (median and mean) of a gene's omics value across cancer types in patient-derived cell line datasets, as implemented in IMOPAC's 'Cancer Types Summary' pancancer expression module, to rank cancer types by expression abundance.

## When to use

Use when a continuous omics value (e.g., mRNA expression) for one gene must be summarized per cancer type across a cell line panel (e.g., DepMap, 1773 lines, 34 cancer types) and ranked to identify the highest-expressing groups.

## When NOT to use

- The goal is a single global statistic over all cell lines rather than per-cancer-type grouping — grouping is essential to the ranking
- The input is drug response or other non-continuous omics better suited to IMOPAC's drug-omics/pathway correlation module rather than the 'Cancer Types Summary' boxplot
- The gene of interest is measured via CNV alteration frequency, where the analogous IMOPAC module reports alteration frequencies, not grouped expression medians

## Inputs

- Gene-by-cell-line mRNA expression matrix from a pancancer cell line dataset (e.g., DepMap: 12 omics, 1773 cell lines, 34 cancer types; GDSC: 992 cell lines, 28 cancer types)
- Cell line metadata mapping each cell line to its cancer type

## Outputs

- Ranked per-cancer-type summary table (median and mean of the gene's expression), e.g., result_tp53_mrna_pancancer.csv
- Boxplot of expression abundance across all cancer types, e.g., tp53_mrna_pancancer_boxplot.svg

## How to apply

Load the pancancer cell line expression matrix (e.g., DepMap transcriptomics covering 1773 cell lines across 34 cancer types) in R and extract the row for the target gene (e.g., TP53). Join the expression vector with cell line metadata that maps each cell line to its cancer type. Using dplyr (v1.0.7 per the IMOPAC repository), group expression by cancer type and compute per-group median and mean; use the median as the ranking statistic, mirroring IMOPAC's 'Cancer Types Summary' module where a boxplot investigates expression abundance of a specific input gene across multiple cancer types. Rank cancer types by median expression in descending order and write the ranked table (e.g., result_tp53_mrna_pancancer.csv) and a boxplot (e.g., tp53_mrna_pancancer_boxplot.svg, SVG.js v2.7 is the rendering format used in IMOPAC). Validate by checking that top-ranked types reproduce known results such as TP53 mRNA being highest in lymphoma, mesothelioma, and lymphoblastic leukaemia in the GDSC database.

## Related tools

- **IMOPAC** (Web server whose 'Cancer Types Summary' module provides the reference pancancer expression boxplot of a specific input gene across multiple cancer types) — https://github.com/wolfgangsk07/IMOPAC
- **dplyr** (R package (v1.0.7) used to group expression by cancer type and compute median/mean summary statistics) — https://github.com/wolfgangsk07/IMOPAC
- **SVG.js** (Library (v2.7) behind the SVG-format figures IMOPAC produces, matching the boxplot output format) — https://github.com/wolfgangsk07/IMOPAC

## Examples

```
Rscript -e 'library(dplyr); expr_df %>% inner_join(meta, by="cell_line") %>% group_by(cancer_type) %>% summarise(median=median(TP53, na.rm=TRUE), mean=mean(TP53, na.rm=TRUE)) %>% arrange(desc(median))'
```

## Evaluation signals

- The per-cancer-type table contains exactly the cancer types present in the metadata, each with finite median and mean (NA handling via na.rm=TRUE)
- Ranking by median places lymphoma, mesothelioma, and lymphoblastic leukaemia among the top cancer types for TP53 mRNA in the GDSC database
- The grouped medians/means fall inside the whisker ranges of the corresponding boxplot for each cancer type
- Output CSV has columns consistent with cancer_type, median, mean and rows sorted by descending median
- Boxplot renders as SVG, matching IMOPAC's figure format

## Limitations

- Summary medians are descriptive; the article does not state statistical testing or multiple-comparison correction for between-type expression comparisons
- Results are specific to the queried database snapshot (GDSC vs DepMap differ: 992 lines/28 types vs 1773 lines/34 types), so top-ranked types may shift between datasets
- The repository README notes the main *.RData datasets cannot be uploaded to GitHub due to file size limits and must be downloaded via urls.txt

## Evidence
