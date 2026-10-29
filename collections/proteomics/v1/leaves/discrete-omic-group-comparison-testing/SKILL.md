---
name: discrete-omic-group-comparison-testing
description: Use when comparing drug sensitivity (AUC/IC50) between two cell-line
  subgroups defined by a discrete molecular alteration (e.g., mutated vs wild-type
  status of a user-defined gene) in GDSC, DepMap, or NCI60 pharmacogenomic data.
when_to_use_negative:
- The omics layer is continuous (mRNA, CNV, methylation, proteomics, metabolomics)
  — use Pearson or Spearman correlation with a coefficient threshold instead of a
  two-group test
- Drugs whose sensitivity values are 'NA' in greater than 15% of the cell lines have
  not yet been excluded — run that filter first or the group test will be underpowered/biased
- The goal is cis-/trans-regulation among omics levels or pathway activity correlation
  (GSVA-based), which are separate IMOPAC modules
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: IMOPAC
  role: Web server hosting the drug–omics correlation module that performs the two-subgroup
    Wilcoxon/Student's t-test comparison of drug responses by discrete omics status
  repo: https://github.com/wolfgangsk07/IMOPAC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:76673b4787153ca2883533b70da5ed7e39f15b5f076ccf3a058ad8a790688c53
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# discrete-omic-group-comparison-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Group-comparison testing used within IMOPAC's drug–omics correlation analysis: for omics with discrete settings (e.g., somatic mutation status), cell lines are split into two subgroups by the mutation status of a user-defined gene and drug response (AUC/IC50) differences are tested with Wilcoxon's sign rank test or Student's t-test, with Benjamini–Hochberg correction across all tested gene–drug pairs.

## When to use

Use when comparing drug sensitivity (AUC/IC50) between two cell-line subgroups defined by a discrete molecular alteration (e.g., mutated vs wild-type status of a user-defined gene) in GDSC, DepMap, or NCI60 pharmacogenomic data.

## When NOT to use

- The omics layer is continuous (mRNA, CNV, methylation, proteomics, metabolomics) — use Pearson or Spearman correlation with a coefficient threshold instead of a two-group test
- Drugs whose sensitivity values are 'NA' in greater than 15% of the cell lines have not yet been excluded — run that filter first or the group test will be underpowered/biased
- The goal is cis-/trans-regulation among omics levels or pathway activity correlation (GSVA-based), which are separate IMOPAC modules

## Inputs

- Drug sensitivity matrix (AUC or IC50) for selected cell lines from GDSC, DepMap, or NCI60, pre-filtered to drop drugs with 'NA' sensitivity in >15% of cell lines
- Discrete multiomics profile (e.g., somatic mutation status) for the user-defined gene across the same cell lines
- User-defined gene identifier and choice of group test (Wilcoxon's sign rank test or Student's t-test)

## Outputs

- Gene–drug association table with group-comparison test statistics, raw P-values, and BH-adjusted P-values for every gene–drug pair tested
- Two-group drug response distributions (mutated vs wild-type subgroups) per tested drug

## How to apply

Load the multiomics profile and drug sensitivity (AUC/IC50) matrices for the selected cell lines from GDSC, DepMap, or NCI60, first excluding any drug whose sensitivity values are 'NA' in greater than 15% of the cell lines. Identify whether the omics layer for the user-defined gene is discrete (e.g., somatic mutation); if so, split the cell models into two subgroups by that gene's mutation status. Apply the chosen group test — Wilcoxon's sign rank test or Student's t-test — to compare drug responses between the mutated and wild-type subgroups. Collect raw P-values across all gene–drug pairs and apply Benjamini–Hochberg adjustment for multiple-hypothesis testing before assembling the gene–drug association table. This discrete-settings branch contrasts with the continuous-settings branch (mRNA, CNV, methylation, proteomics, metabolomics), which instead uses Pearson or Spearman correlation against a user-defined coefficient threshold.

## Related tools

- **IMOPAC** (Web server hosting the drug–omics correlation module that performs the two-subgroup Wilcoxon/Student's t-test comparison of drug responses by discrete omics status) — https://github.com/wolfgangsk07/IMOPAC

## Evaluation signals

- Output association table contains raw P-values and Benjamini–Hochberg-adjusted P-values for every gene–drug pair tested
- No drug appears in the results whose sensitivity values were 'NA' in greater than 15% of the cell lines
- Subgroups partition the cell models into exactly two groups defined by the user-defined gene's mutation status (mutated vs wild-type)
- Rows failing the user-defined significance criterion are excluded, mirroring how coefficient-threshold rows are dropped in the continuous-omics branch
- Test type matches the chosen setting: Wilcoxon's sign rank test or Student's t-test, not a correlation coefficient

## Limitations

- IMOPAC's drug response predictive model, which shares the GDSC data foundation, has not yet been validated on an independent dataset
- The predictive model may be affected by other factors beyond the omics features used
- PDX and organoid multiomics and drug screen cohorts are not yet integrated, so discrete-omic comparisons are limited to GDSC, DepMap, and NCI60 cell-line data

## Evidence
