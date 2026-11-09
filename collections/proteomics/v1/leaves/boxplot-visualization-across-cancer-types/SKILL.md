---
name: boxplot-visualization-across-cancer-types
description: Use when you need to rank or compare the expression abundance of a single
  gene (e.g., TP53 mRNA) across many cancer types from a pancancer cell line expression
  matrix, or when reproducing IMOPAC-style Cancer Types Summary plots.
when_to_use_negative:
- The analysis goal is drug-omics correlation or drug response rather than expression
  abundance of a single gene across cancer types — IMOPAC provides separate 'Drug-Omics
  Correlation' and 'Drug Response Prediction' modules for those.
- The input is already aggregated per cancer type (summary statistics) rather than
  per-cell-line expression values, so a boxplot of the underlying distribution cannot
  be drawn.
- The variable of interest is categorical rather than continuous — IMOPAC's Cancer
  Types Summary boxplot is specifically for continuous settings such as mRNA.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: IMOPAC
  role: Web server implementing the 'Cancer Types Summary' module that draws the pancancer
    expression boxplot for a user-input gene across cancer types
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: DepMap
  role: Source of pancancer cell line expression data (12 omics, 1773 cell lines from
    34 cancer types) used as input to the boxplot analysis
- name: GDSC
  role: Alternative source database of cell line omics (992 cell lines from 28 cancer
    types) on which the TP53 mRNA pancancer boxplot finding was reported
- name: SVG.js
  role: JavaScript library (v2.7) used by IMOPAC to generate the SVG-format figures
    shown on the website
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:76b36681007933ffa73e87eb13a8ef17145f78e4514d3a2cb54ce0e486b69c2c
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
  - GDSC
  - IMOPAC
  - R
  - SVG.js
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# boxplot-visualization-across-cancer-types

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pancancer expression visualization in the style of IMOPAC's 'Cancer Types Summary' module: a boxplot is used to investigate the expression abundance (e.g., mRNA) of a specific input gene across multiple cancer types in patient-derived cancer cell line datasets such as GDSC and DepMap.

## When to use

Use when you need to rank or compare the expression abundance of a single gene (e.g., TP53 mRNA) across many cancer types from a pancancer cell line expression matrix, or when reproducing IMOPAC-style Cancer Types Summary plots.

## When NOT to use

- The analysis goal is drug-omics correlation or drug response rather than expression abundance of a single gene across cancer types — IMOPAC provides separate 'Drug-Omics Correlation' and 'Drug Response Prediction' modules for those.
- The input is already aggregated per cancer type (summary statistics) rather than per-cell-line expression values, so a boxplot of the underlying distribution cannot be drawn.
- The variable of interest is categorical rather than continuous — IMOPAC's Cancer Types Summary boxplot is specifically for continuous settings such as mRNA.

## Inputs

- pancancer cell line mRNA/expression matrix (e.g., DepMap transcriptomics, 1773 cell lines x 34 cancer types; or GDSC, 992 cell lines x 28 cancer types)
- cell line metadata table mapping each cell line to its cancer type
- gene identifier of interest (e.g., TP53)

## Outputs

- boxplot (SVG) of gene expression abundance across cancer types
- ranked summary table of median/mean expression per cancer type (e.g., result_tp53_mrna_pancancer.csv)

## How to apply

Load a pancancer cell line mRNA expression matrix (e.g., DepMap transcriptomic data covering 1773 cell lines across 34 cancer types, or GDSC with 992 cell lines from 28 cancer types) in R. Extract the row for the input gene (e.g., TP53) and join it with cell line metadata mapping each cell line to its cancer type. Group expression by cancer type and compute per-type summary statistics (median and mean) with dplyr, then draw a boxplot of expression abundance across all cancer types as IMOPAC's 'Cancer Types Summary' module does for continuous omics settings. Rank cancer types by median expression in descending order and export the ranked table (e.g., result_tp53_mrna_pancancer.csv) and the figure (e.g., tp53_mrna_pancancer_boxplot.svg, the format IMOPAC generates with SVG.js). Validate by checking that top-ranked cancer types reproduce the reported result (TP53 mRNA highest in lymphoma, mesothelioma, and lymphoblastic leukaemia in the GDSC database).

## Related tools

- **IMOPAC** (Web server implementing the 'Cancer Types Summary' module that draws the pancancer expression boxplot for a user-input gene across cancer types) — https://github.com/wolfgangsk07/IMOPAC
- **DepMap** (Source of pancancer cell line expression data (12 omics, 1773 cell lines from 34 cancer types) used as input to the boxplot analysis)
- **GDSC** (Alternative source database of cell line omics (992 cell lines from 28 cancer types) on which the TP53 mRNA pancancer boxplot finding was reported)
- **SVG.js** (JavaScript library (v2.7) used by IMOPAC to generate the SVG-format figures shown on the website)

## Examples

```
Rscript -e 'boxplot(TP53 ~ cancer_type, data = tp53_df, ylab = "TP53 mRNA expression"); write.csv(arrange(summarise(group_by(tp53_df, cancer_type), median = median(TP53))), "result_tp53_mrna_pancancer.csv")'
```

## Evaluation signals

- The ranked summary table places lymphoma, mesothelioma, and lymphoblastic leukaemia among the top cancer types by median TP53 mRNA level (as reported for the GDSC database, Fig. 2B).
- The output figure is an SVG file (IMOPAC generates website figures in svg format via SVG.js v2.7).
- The boxplot shows one box per cancer type covering all cancer types present in the input matrix (e.g., 34 cancer types for DepMap, 28 for GDSC).
- Per-cancer-type median and mean summary statistics are computable from the grouped dplyr table and consistent with the plotted distributions.

## Limitations

- Dataset-specific results differ by source: the reported TP53 ranking corresponds to the GDSC database and may not hold identically in DepMap or NCI60 (NCI60 covers only 60 cell lines from 9 cancer types).
- The .RData datasets cannot be hosted in the GitHub repository due to file size limits, so users must download them separately via urls.txt to reproduce analyses locally.

## Evidence

- [methods] Pancancer expression analysis: "a boxplot is used to investigate the expression abundance for those with continuous settings (e.g., mRNA and so on) of a specific input gene across multiple cancer types in the “Cancer Types Summary”"
- [results] TP53 pancancer finding: "the mRNA level of TP53 is the highest in lymphoma, mesothelioma, and lymphoblastic leukaemia in the GDSC database (Fig. 2B)"
- [methods] Dataset scales: "12 omics, 1773 cell lines from 34 cancer types and 22 drugs are available in DepMap database"
- [methods] SVG figure output: "Figures shown on the website are in the svg format generated by SVG.js (v2.7, https://svgjs.dev/docs/2.7/)"
- [readme] README dataset hosting limitation: "due to the file size limitation of github, the dataset( mainly *.RData ) can't be uploaded here. You can get the urls of all RDatas in urls.txt."
