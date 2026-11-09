---
name: msigdb-geneset-mapping
description: Use when a gene-by-sample expression matrix (e.g. DepMap/GDSC mRNA values
  for cancer cell lines) must be converted into pathway-level activity scores using
  MSigDB or user-defined gene sets to support drug-pathway correlation.
when_to_use_negative:
- Input is already a pathway-activity score matrix — proceed directly to drug-pathway
  correlation instead of re-scoring
- The analysis goal is single-gene drug-omics correlation (IMOPAC's drug-omics correlation
  module handles molecule-level Pearson/Spearman without GSVA)
- The omics data are non-transcriptomic (e.g. methylation, CNV, proteomics) — GSVA
  here is applied only to a transcriptomic geneset
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSVA
  role: R package (v1.40.1) that computes gene set variation analysis enrichment scores,
    converting the gene-by-cell-line expression matrix into a pathway-by-cell-line
    activity matrix
- name: MSigDB
  role: Source of the pathway gene set definitions used as the transcriptomic genesets
    for GSVA scoring
- name: impute
  role: R package (v1.66.0) providing KNN imputation of missing values in the expression
    matrix prior to GSVA scoring
- name: IMOPAC
  role: Web server whose Drug-Pathway module consumes the pathway activity score matrix
    to correlate pathway activity with drug response across cell lines
  repo: https://github.com/wolfgangsk07/IMOPAC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:c13ebc3345d6bbd5c640c04355590f473ec973761d190b3cc261fc298fa4054c
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
  - GSVA
  - IMOPAC
  - MSigDB
  - impute
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# msigdb-geneset-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps MSigDB (or user-customized) gene sets onto a gene-by-cell-line transcriptomic expression matrix and applies gene set variation analysis (GSVA) to produce a pathway-activity score matrix, as implemented in IMOPAC's Drug-Pathway module for correlating pathway activity with drug response.

## When to use

Use when a gene-by-sample expression matrix (e.g. DepMap/GDSC mRNA values for cancer cell lines) must be converted into pathway-level activity scores using MSigDB or user-defined gene sets to support drug-pathway correlation.

## When NOT to use

- Input is already a pathway-activity score matrix — proceed directly to drug-pathway correlation instead of re-scoring
- The analysis goal is single-gene drug-omics correlation (IMOPAC's drug-omics correlation module handles molecule-level Pearson/Spearman without GSVA)
- The omics data are non-transcriptomic (e.g. methylation, CNV, proteomics) — GSVA here is applied only to a transcriptomic geneset

## Inputs

- Gene-by-cell-line mRNA expression matrix from DepMap/GDSC (missing values imputed with KNN via the impute R package)
- MSigDB gene set definitions (or a user-customized transcriptomic gene set)

## Outputs

- Pathway-by-cell-line GSVA activity score matrix (pathway_activity_scores.csv) with one column per pathway and one row per cell line

## How to apply

Load the transcriptomic gene expression matrix of cancer cell lines from the DepMap/GDSC pharmacogenomic datasets, imputing missing values with KNN via the impute R package before scoring. Load the MSigDB gene sets (or a user-customized transcriptomic gene set) as the pathway definitions, and run gene set variation analysis with the GSVA R package v1.40.1 to convert the gene-by-cell-line matrix into a pathway-by-cell-line activity score matrix. Write the result to pathway_activity_scores.csv so the downstream IMOPAC drug-pathway correlation analysis (Pearson/Spearman) can relate pathway activity to drug response. The rationale is that GSVA summarizes coordinate expression of gene sets into per-sample enrichment scores, enabling pathway-level (rather than single-gene) pharmacogenomic associations.

## Related tools

- **GSVA** (R package (v1.40.1) that computes gene set variation analysis enrichment scores, converting the gene-by-cell-line expression matrix into a pathway-by-cell-line activity matrix)
- **MSigDB** (Source of the pathway gene set definitions used as the transcriptomic genesets for GSVA scoring)
- **impute** (R package (v1.66.0) providing KNN imputation of missing values in the expression matrix prior to GSVA scoring)
- **IMOPAC** (Web server whose Drug-Pathway module consumes the pathway activity score matrix to correlate pathway activity with drug response across cell lines) — https://github.com/wolfgangsk07/IMOPAC

## Examples

```
library(GSVA); pathway_scores <- gsva(expr_matrix, msigdb_genesets)
```

## Evaluation signals

- Output matrix has exactly one column per MSigDB pathway
- Output matrix has exactly one row per cell line, matching the input expression matrix's samples
- Every cell in the activity matrix contains a finite numeric GSVA enrichment score
- Imputed expression matrix contains no missing values before GSVA is run
- Resulting pathway_activity_scores.csv feeds valid Pearson/Spearman correlations against GDSC/DepMap drug sensitivity values

## Limitations

- Pathway definitions are restricted to MSigDB or user-customized gene sets; no other pathway databases are supported in this module
- The input expression matrix must have missing values imputed (KNN via impute) before GSVA; raw matrices with NAs will fail
- The IMOPAC repository README notes that the dataset .RData files cannot be uploaded to GitHub due to file size limits and must be fetched from the URLs in urls.txt before the pipeline can run

## Evidence

- [readme] GSVA package version in runtime: [evidence span withheld — anchor status: not_found]
