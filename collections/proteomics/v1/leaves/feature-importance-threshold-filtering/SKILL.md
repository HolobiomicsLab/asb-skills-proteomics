---
name: feature-importance-threshold-filtering
description: Use when you need to enumerate multiple candidate biomarker signatures
  from a labeled omics dataset (e.g. proteomics matrix) without exploring all feature
  combinations, by ranking features with Elastic Net and applying an FI cutoff.
when_to_use_negative:
- When you need a single optimal signature and exhaustive combination search is computationally
  feasible
- When the input lacks a binary class label, since signatures are scored for predictive
  performance (AUC via pROC)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: muSignAl
  role: Runs the FI-threshold-based enumeration of multiple signatures with similar
    predictive performance
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits regularized (Elastic Net alpha = 0.9) models that rank features by importance
    for the k = 0.9 cutoff
- name: pROC
  role: Scores each enumerated signature's predictive performance as an AUC value
- name: devtools
  role: Installs muSignAl from GitHub via devtools::install_github("ShuklaLab/muSignAl")
- name: R v3.6
  role: Runtime environment in which muSignAl was coded and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:012419ca2d872efd658acebf8547d6d264e7725190e796dd788bc95fcae2b81d
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Feature-importance threshold filtering

## Summary

An approach from muSignAl that uses a feature-importance (FI) cutoff (e.g. k = 0.9) on glmnet-ranked features to enumerate multiple candidate omic signatures with similar predictive performance, instead of exhaustively testing all feature combinations.

## When to use

Use when you need to enumerate multiple candidate biomarker signatures from a labeled omics dataset (e.g. proteomics matrix) without exploring all feature combinations, by ranking features with Elastic Net and applying an FI cutoff.

## When NOT to use

- When you need a single optimal signature and exhaustive combination search is computationally feasible
- When the input lacks a binary class label, since signatures are scored for predictive performance (AUC via pROC)

## Inputs

- Labeled omics feature matrix (e.g. AD.xlsx, Olink Proteomics CVD-II data with 77 patient samples x 91 protein features)
- Feature-importance threshold k (e.g. 0.9)
- Elastic Net alpha parameter (0.9 in the reference run)

## Outputs

- Set of enumerated candidate signatures with their protein-feature members
- Per-signature AUC values computed with pROC
- Signature table exported as an Excel file (e.g. sign_cvd2.xlsx)

## How to apply

Fit regularized models with glmnet (Elastic Net alpha = 0.9) to rank protein or other omic features by importance, then apply a feature-importance threshold (k = 0.9 in the reference run) as the cutoff that defines which features can enter each candidate signature. Use muSignAl() to enumerate candidate signatures under this cutoff — for the Brunner et al. Olink CVD-II dataset (AD.xlsx, 77 samples x 91 protein features), this produced 1984 signatures, of which 158 were unique. Score each returned signature with pROC to obtain per-signature AUC values and retain those meeting a performance criterion (e.g. AUC > 0.95, which held for 47 unique signatures). Export the signature set with its per-signature AUCs (e.g. as sign_cvd2.xlsx) to compare against the reference output. The rationale is that the FI threshold systematically bypasses exhaustive feature-combination search while yielding multiple signatures of similar predictive performance.

## Related tools

- **muSignAl** (Runs the FI-threshold-based enumeration of multiple signatures with similar predictive performance) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits regularized (Elastic Net alpha = 0.9) models that rank features by importance for the k = 0.9 cutoff)
- **pROC** (Scores each enumerated signature's predictive performance as an AUC value)
- **devtools** (Installs muSignAl from GitHub via devtools::install_github("ShuklaLab/muSignAl"))
- **R v3.6** (Runtime environment in which muSignAl was coded and executed)

## Examples

```
install.packages(c("glmnet","pROC")); devtools::install_github("ShuklaLab/muSignAl"); Rscript test.R
```

## Evaluation signals

- Running the k = 0.9 filter on AD.xlsx (77 samples x 91 features) returns 1984 signatures, of which 158 are unique
- 47 of the unique signatures achieve AUC greater than 0.95 when scored with pROC
- Exported signature table matches the reference output sign_cvd2.xlsx from the repository's example folder
- Every signature returned contains only features whose importance meets or exceeds the k = 0.9 cutoff

## Limitations

- Threshold calibration is dataset-dependent; k = 0.9 is the reference value validated on the Brunner et al. Olink CVD-II proteomics dataset
- Requires R v3.6 with glmnet and pROC pre-installed, and muSignAl installed via devtools from GitHub
- The FI cutoff reduces but does not replace downstream AUC-based screening of the enumerated signatures

## Evidence

- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [abstract] selects multiple signatures with similar predictive performance while bypassing exhaustive search: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
- [readme] README example folder contents: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder."
- [results] Reference-run result on the Brunner dataset: [evidence span withheld — anchor status: not_found]
