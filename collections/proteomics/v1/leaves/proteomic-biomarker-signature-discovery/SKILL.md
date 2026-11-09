---
name: proteomic-biomarker-signature-discovery
description: Use when a labelled omics dataset (e.g., proteomics .xlsx with case/control
  outcomes) needs multiple equally predictive feature signatures without exhaustively
  testing all feature combinations.
when_to_use_negative:
- When an exhaustive enumeration of all feature combinations is explicitly required
  — muSignAl 'systematically bypass[es] the requirement of exploring all the combinations
  of features'.
- When the input table lacks binary phenotype labels needed for glmnet regularized
  fitting and pROC AUC scoring.
- When a single best signature suffices and multiple equally performing alternatives
  offer no biomarker-development-cost benefit.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3517
tools:
- name: muSignAl
  role: Core algorithm that enumerates multiple signatures with similar predictive
    performance given an FI threshold k
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits Elastic Net regularized models (alpha = 0.9) to rank protein features
    by importance
- name: pROC
  role: Computes per-signature AUC from the glmnet-fitted models to filter signatures
    with AUC > 0.95
- name: devtools
  role: Installs the muSignAl package from the ShuklaLab GitHub repository
  repo: https://github.com/ShuklaLab/muSignAl
- name: R v3.6
  role: Language/runtime in which muSignAl is coded and executed
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:9869296334fda653a6fe96a9674b653ae8205b08feb58116fdb2f5d1c6a03fa5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - R v3.6
  - devtools
  - glmnet
  - muSignAl
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# proteomic-biomarker-signature-discovery

## Summary

Discover multiple omic biomarker signatures with similar predictive performance using muSignAl, an R algorithm that combines Elastic Net (glmnet) feature-importance ranking with a threshold k to enumerate candidate signatures, then scores each with pROC AUC. Applied here to the Brunner et al. Olink CVD-II proteomics dataset (77 samples × 91 proteins).

## When to use

Use when a labelled omics dataset (e.g., proteomics .xlsx with case/control outcomes) needs multiple equally predictive feature signatures without exhaustively testing all feature combinations.

## When NOT to use

- When an exhaustive enumeration of all feature combinations is explicitly required — muSignAl 'systematically bypass[es] the requirement of exploring all the combinations of features'.
- When the input table lacks binary phenotype labels needed for glmnet regularized fitting and pROC AUC scoring.
- When a single best signature suffices and multiple equally performing alternatives offer no biomarker-development-cost benefit.

## Inputs

- Labelled omics feature table in Excel format, e.g. AD.xlsx (Brunner et al. Olink CVD-II proteomics, 77 samples × 91 protein features)
- Feature-importance threshold k (e.g., k = 0.9)
- Elastic Net mixing parameter alpha (e.g., alpha = 0.9)

## Outputs

- Set of candidate signatures with per-signature AUC values (1984 signatures, 158 unique at k = 0.9)
- Filtered shortlist of signatures with AUC > 0.95 (47 signatures, e.g., filtered_signatures_auc_gt_095.xlsx)
- Reference output file sign_cvd2.xlsx matching the repository example folder

## How to apply

Install R v3.6 with glmnet and pROC pre-installed, then install muSignAl from GitHub via devtools::install_github("ShuklaLab/muSignAl"). Load the labelled feature table (e.g., AD.xlsx, 77 patient samples × 91 Olink protein features) and run the muSignAl() function with a feature-importance (FI) threshold k = 0.9 and Elastic Net alpha = 0.9, so glmnet ranks protein features by importance and the k cutoff enumerates multiple candidate signatures without exploring all feature combinations. Score each returned signature's predictive performance with pROC to obtain per-signature AUC values. Export the signature set with per-signature AUCs (e.g., sign_cvd2.xlsx) and filter to signatures with AUC > 0.95 to build a Table-1-style shortlist of equally good panels for cost-optimised biomarker development.

## Related tools

- **muSignAl** (Core algorithm that enumerates multiple signatures with similar predictive performance given an FI threshold k) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits Elastic Net regularized models (alpha = 0.9) to rank protein features by importance)
- **pROC** (Computes per-signature AUC from the glmnet-fitted models to filter signatures with AUC > 0.95)
- **devtools** (Installs the muSignAl package from the ShuklaLab GitHub repository) — https://github.com/ShuklaLab/muSignAl
- **R v3.6** (Language/runtime in which muSignAl is coded and executed)

## Examples

```
devtools::install_github("ShuklaLab/muSignAl")
```

## Evaluation signals

- At k = 0.9 on the Brunner et al. dataset, the run should yield 1984 signatures of which 158 are unique.
- Exactly 47 unique signatures should pass the AUC > 0.95 filter, reproducing Table 1 of the article.
- The top-ranked signature (no. 150: CCL17/DECR1/ADM/PARP-1/SORT1) should reach AUC = 1.00.
- The exported output file should match the reference sign_cvd2.xlsx shipped in the package's example folder.
- The repository test script test.R run on AD.xlsx should execute end-to-end without errors on R v3.6 with glmnet and pROC installed.

## Limitations

- Requires R v3.6 with glmnet and pROC pre-installed before installation.
- The workflow was demonstrated only on a proteomics dataset; extension to other omics types is proposed but not validated here.
- The k threshold controls candidate enumeration — it bypasses exhaustive combination search, so signatures outside the importance cutoff are never considered.

## Evidence

- [abstract] we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance: "we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance"
- [abstract] selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [methods] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [readme] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed, which can be installed as below: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed, which can be installed as below"
