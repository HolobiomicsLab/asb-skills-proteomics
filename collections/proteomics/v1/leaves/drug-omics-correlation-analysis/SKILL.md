---
name: drug-omics-correlation-analysis
description: Use when you want to identify gene–drug associations from multiomics
  and drug sensitivity matrices of GDSC, DepMap, or NCI60 cell lines in IMOPAC, for
  a user-defined gene of interest.
when_to_use_negative:
- The input omics is a fusion query, molecular classification, or pancancer expression
  analysis — IMOPAC has separate modules for those and this skill is not the right
  choice
- The user wants to correlate drug response with pathway activity computed by GSVA
  — the drug-pathway correlation analysis is a separate module
- The cell line set has fewer than ~85% drugs with sensitivity values, i.e. most drugs
  would be excluded by the 15% NA cutoff, leaving no usable associations
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3517
tools:
- name: IMOPAC
  role: Web server whose Drug-Omics module implements the user-defined correlation
    analysis (Pearson/Spearman) between multiomics molecular values and drug response,
    with the 15% NA drug filter and BH adjustment
  repo: https://github.com/wolfgangsk07/IMOPAC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:ce56084ea06a760e31ee4cbd932dddd7f29c3a6b8c769f54bc2e84dd46f57e54
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# drug-omics-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes associations between the multiomics profile (e.g. mRNA, CNV, methylation, proteomics, metabolomics, somatic mutation) of a user-defined gene and drug sensitivity (AUC/IC50) across patient-derived cancer cell lines in IMOPAC, using Pearson/Spearman correlation for continuous omics and group tests for discrete omics.

## When to use

Use when you want to identify gene–drug associations from multiomics and drug sensitivity matrices of GDSC, DepMap, or NCI60 cell lines in IMOPAC, for a user-defined gene of interest.

## When NOT to use

- The input omics is a fusion query, molecular classification, or pancancer expression analysis — IMOPAC has separate modules for those and this skill is not the right choice
- The user wants to correlate drug response with pathway activity computed by GSVA — the drug-pathway correlation analysis is a separate module
- The cell line set has fewer than ~85% drugs with sensitivity values, i.e. most drugs would be excluded by the 15% NA cutoff, leaving no usable associations

## Inputs

- Multiomics expression matrices (mRNA, CNV, methylation, proteomics, metabolomics, somatic mutation) for a user-defined gene across cell lines
- Drug sensitivity matrices (AUC or IC50) for GDSC, DepMap, or NCI60 cell lines
- User-selected correlation method (Pearson or Spearman) and correlation-coefficient threshold
- For discrete omics (e.g. somatic mutation): mutation status of the user-defined gene per cell line

## Outputs

- Gene–drug association table containing correlation coefficients, raw P-values, and BH-adjusted P-values for every gene–drug pair tested
- Filtered set of associations passing the user-defined correlation-coefficient threshold

## How to apply

Load the multiomics profiles and drug sensitivity (AUC/IC50) matrices for the selected cell lines from the GDSC, DepMap, or NCI60 dataset. First exclude drugs whose sensitivity values are 'NA' in greater than 15% of the cell lines. If the omics has a discrete setting (e.g. somatic mutation), split the cell models into two subgroups by the mutation status of the user-defined gene and apply Wilcoxon's sign rank test or Student's t-test to compare drug responses between subgroups; if the omics has a continuous setting (e.g. mRNA, CNV, methylation, proteomics, metabolomics), compute the user-selected Pearson or Spearman correlation between molecular values and drug response, retaining associations passing the user-defined correlation-coefficient threshold. Apply Benjamini–Hochberg adjustment to all P-values for multiple-hypothesis testing and assemble the results into a gene–drug association table. The rationale is that the 15% NA cutoff keeps only drugs with sufficient sensitivity coverage, and BH adjustment controls false discovery across the many gene–drug pairs tested.

## Related tools

- **IMOPAC** (Web server whose Drug-Omics module implements the user-defined correlation analysis (Pearson/Spearman) between multiomics molecular values and drug response, with the 15% NA drug filter and BH adjustment) — https://github.com/wolfgangsk07/IMOPAC

## Evaluation signals

- The output table contains columns for correlation coefficient, raw P-value, and BH-adjusted P-value for every gene–drug pair tested
- Rows whose correlation coefficient falls below the user-defined threshold are absent from the final table
- No drug appearing in the output has sensitivity values 'NA' in greater than 15% of the cell lines
- For discrete omics (e.g. somatic mutation), the output reports group-test results (Wilcoxon or Student's t-test) rather than correlation coefficients
- All reported P-values are BH-adjusted for multiple-hypothesis testing

## Limitations

- Drugs whose sensitivity values are 'NA' in greater than 15% of the cell lines are excluded, so drugs with sparse coverage cannot be analysed
- Associations are limited to the user-defined gene; genome-wide screening is outside this module's scope
- The predictive-model discussion notes the model 'may be affected by other factors' and 'further validation on an independent dataset' is needed, so the same caution applies to any derived gene–drug association

## Evidence
