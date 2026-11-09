---
name: biomarker-signature-thresholding
description: Use when building an omic biomarker signature from a feature/sample matrix
  and you need a stability-based cutoff (feature frequency across resampled models)
  rather than a single-model coefficient ranking.
when_to_use_negative:
- The goal is to exhaustively explore all combinations of features — muSignAl is explicitly
  designed to bypass that requirement while finding multiple signatures with similar
  predictive performance
- The fitted subsample models do not exceed chance performance (AUC <= 0.5) — such
  models are discarded and provide no stable features to threshold
- A single deterministic model's coefficient ranking is sufficient, rather than resampling-based
  feature stability across 100 subsamples
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: muSignAl
  role: R package implementing the multiple-signature algorithm whose feature-selection
    stage computes FI and applies the k threshold
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits the Elastic Net generalized linear models (alpha = 0.9) on each 80% subsample
    used to compute feature appearance frequency
- name: pROC
  role: Computes the AUC used to validate and discard subsample models with AUC not
    greater than 0.5
- name: devtools
  role: Used to install muSignAl from GitHub via devtools::install_github
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:b8b180dc6c1c25d4bfc19d458cb8e6ee39759bfd54e1cf5e83127558db85e822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - devtools
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# biomarker-signature-thresholding

## Summary

Threshold-based biomarker feature selection as implemented in the muSignAl R package: feature importance (FI) is defined as the proportion of retained Elastic Net subsample models in which a feature appears, and features are kept when FI >= k (default 0.9) to form the first signature.

## When to use

Use when building an omic biomarker signature from a feature/sample matrix and you need a stability-based cutoff (feature frequency across resampled models) rather than a single-model coefficient ranking.

## When NOT to use

- The goal is to exhaustively explore all combinations of features — muSignAl is explicitly designed to bypass that requirement while finding multiple signatures with similar predictive performance
- The fitted subsample models do not exceed chance performance (AUC <= 0.5) — such models are discarded and provide no stable features to threshold
- A single deterministic model's coefficient ranking is sufficient, rather than resampling-based feature stability across 100 subsamples

## Inputs

- Sample-by-feature omic matrix in Excel format (e.g. AD.xlsx from the muSignAl example folder)
- R v3.6 environment with glmnet and pROC pre-installed

## Outputs

- Selected first-set feature list of features with FI >= 0.9
- Final output signature file (e.g. sign_cvd2.xlsx)

## How to apply

Load the sample dataset (e.g. AD.xlsx) into R v3.6, generate 100 random subsamples each holding 80% of samples, and fit an Elastic Net generalized linear model with glmnet on each subsample using alpha = 0.9 (90% LASSO / 10% Ridge penalty mix). Evaluate each fitted model with pROC to obtain the AUC and discard any model whose AUC is not greater than 0.5. For each candidate feature, compute feature importance as the proportion of retained models in which that feature appears, and keep features with FI >= k, where the default threshold k = 0.9. The selected first-set features then have their predictive performance calculated, and results are exported (e.g. to sign_cvd2.xlsx).

## Related tools

- **muSignAl** (R package implementing the multiple-signature algorithm whose feature-selection stage computes FI and applies the k threshold) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits the Elastic Net generalized linear models (alpha = 0.9) on each 80% subsample used to compute feature appearance frequency)
- **pROC** (Computes the AUC used to validate and discard subsample models with AUC not greater than 0.5)
- **devtools** (Used to install muSignAl from GitHub via devtools::install_github)

## Examples

```
install.packages(c("glmnet","pROC")); devtools::install_github("ShuklaLab/muSignAl"); test.R  # runs on example/AD.xlsx, writes sign_cvd2.xlsx
```

## Evaluation signals

- Every retained subsample model has AUC > 0.5 as evaluated with pROC
- Every exported feature has feature importance FI >= 0.9 (default threshold k)
- Feature importance values are proportions of retained models (bounded in [0,1]) in which the feature appears
- Subsampling used 100 random subsamples each holding 80% of the samples, with glmnet alpha = 0.9
- Final output written as sign_cvd2.xlsx matching the example-folder structure

## Limitations

- Requires glmnet and pROC to be pre-installed and was coded/tested in R v3.6
- Models with AUC not greater than 0.5 are discarded, so on weak-signal datasets few or no models survive and FI cannot be computed reliably
- The default threshold k = 0.9 is a stability cutoff; features appearing in fewer than 90% of retained models are excluded from the first signature

## Evidence

- [methods] Feature importance definition and threshold: [evidence span withheld — anchor status: not_found]
- [results] FI threshold selection rule: [evidence span withheld — anchor status: not_found]
- [abstract] Algorithm purpose: "we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance"
- [readme] Toolchain and version requirements: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [methods] Example run artifacts: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
