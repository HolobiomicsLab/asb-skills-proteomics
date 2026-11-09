---
name: pancancer-expression-analysis
description: Use when you need to compare a single gene's expression (e.g., mRNA,
  CNV, methylation) across all cancer types in a pancancer cell-line dataset to identify
  which cancer types show the highest or lowest levels.
when_to_use_negative:
- The question concerns gene-drug associations or correlations rather than expression
  distribution across cancer types — use IMOPAC's drug-omics/pathway correlation analysis
  instead.
- The analysis targets a single cancer type's molecular subtypes — use IMOPAC's cancer
  subtype (unsupervised clustering) analysis instead.
- The input is drug sensitivity data with >15% 'NA' values across cell lines, which
  the article's filtering excludes from pharmacogenomic analyses.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: IMOPAC
  role: Web server providing the 'Cancer Types Summary' module that renders pancancer
    expression boxplots for a user-input gene across cancer types
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: R
  role: Used for loading the expression matrix, extracting the gene row, joining cancer-type
    metadata, and computing dplyr summary statistics and ranked tables
- name: DepMap
  role: Source database of pancancer cell-line multiomics data (12 omics, 1773 cell
    lines, 34 cancer types) used as the expression matrix input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:c0ee2813160392750a65333714bcad61eb71cf17a22529d022561f4013965c1f
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
  - DepMap
  - IMOPAC
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pancancer-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks cancer types by the expression abundance of a user-input gene across a pancancer cell-line expression matrix (e.g., DepMap: 12 omics, 1773 cell lines, 34 cancer types), visualized as a boxplot in IMOPAC's 'Cancer Types Summary' module.

## When to use

Use when you need to compare a single gene's expression (e.g., mRNA, CNV, methylation) across all cancer types in a pancancer cell-line dataset to identify which cancer types show the highest or lowest levels.

## When NOT to use

- The question concerns gene-drug associations or correlations rather than expression distribution across cancer types — use IMOPAC's drug-omics/pathway correlation analysis instead.
- The analysis targets a single cancer type's molecular subtypes — use IMOPAC's cancer subtype (unsupervised clustering) analysis instead.
- The input is drug sensitivity data with >15% 'NA' values across cell lines, which the article's filtering excludes from pharmacogenomic analyses.

## Inputs

- Pancancer cancer cell line expression matrix (e.g., DepMap transcriptomic RData/CSV covering 1773 cell lines x 34 cancer types)
- Cell line metadata mapping cell line identifiers to cancer types
- Gene identifier for the query gene (e.g., TP53)

## Outputs

- Ranked table of cancer types by median (and mean) gene expression (e.g., result_tp53_mrna_pancancer.csv)
- Boxplot of gene expression abundance across cancer types (e.g., tp53_mrna_pancancer_boxplot.svg)

## How to apply

Load the pancancer expression matrix (e.g., the DepMap transcriptomic dataset covering 1773 cell lines across 34 cancer types) and extract the row for the input gene (e.g., TP53). Join the gene expression values with cell line metadata mapping each cell line to its cancer type, then group by cancer type and compute per-type summary statistics (median and mean) with dplyr. Visualize expression abundance as a boxplot across all cancer types, matching the 'Cancer Types Summary' module of IMOPAC, which uses boxplots for continuous settings such as mRNA. Finally, rank cancer types by median expression in descending order to surface the top cancer types; for TP53 in the GDSC data this yields lymphoma, mesothelioma, and lymphoblastic leukaemia as the highest.

## Related tools

- **IMOPAC** (Web server providing the 'Cancer Types Summary' module that renders pancancer expression boxplots for a user-input gene across cancer types) — https://github.com/wolfgangsk07/IMOPAC
- **R** (Used for loading the expression matrix, extracting the gene row, joining cancer-type metadata, and computing dplyr summary statistics and ranked tables)
- **DepMap** (Source database of pancancer cell-line multiomics data (12 omics, 1773 cell lines, 34 cancer types) used as the expression matrix input)

## Examples

```
Rscript -e 'library(dplyr); expr <- read.csv("depmap_expr.csv"); meta <- read.csv("cellline_meta.csv"); expr %>% inner_join(meta, by="cell_line") %>% filter(gene=="TP53") %>% group_by(cancer_type) %>% summarise(median_expr=median(expression)) %>% arrange(desc(median_expr))'
```

## Evaluation signals

- The top-ranked cancer types by median TP53 mRNA level should reproduce the reported result: lymphoma, mesothelioma, and lymphoblastic leukaemia (GDSC database, Fig. 2B).
- The ranked summary table (e.g., result_tp53_mrna_pancancer.csv) contains one row per cancer type with median and mean expression values and is sorted in descending order of median.
- The boxplot output renders expression abundance for every cancer type represented in the matrix (up to 34 cancer types for DepMap data).
- Every cell line in the expression matrix is mapped to exactly one cancer type after the metadata join; unmatched cell lines should be accounted for.
- The figure is generated in SVG format, consistent with IMOPAC's figure rendering.

## Limitations

- The article reports the TP53 result for the GDSC database; results may differ across databases (GDSC: 992 cell lines/28 cancer types vs. DepMap: 1773 cell lines/34 cancer types).
- The analysis reflects cell line models only, not patient tumours; IMOPAC's planned PDX and organoid cohorts are not yet integrated.
- Median-based ranking can obscure within-cancer-type heterogeneity and is sensitive to cancer types with few cell lines.

## Evidence
