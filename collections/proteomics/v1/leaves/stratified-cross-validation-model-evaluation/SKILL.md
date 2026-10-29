---
name: stratified-cross-validation-model-evaluation
description: Use when a user-defined multiomics signature (e.g. the 15-gene hypoxia
  signature) must be tested for its ability to predict a user-defined drug response
  (e.g. Nilotinib) from GDSC cell-line data with an AUC-quantified ML evaluation.
when_to_use_negative:
- Input is already a trained model or predicted-response vector — this skill only
  builds and cross-validates models from raw GDSC drug sensitivity plus multiomics
  features
- The dataset is not one of the IMOPAC-integrated pharmacogenomic databases (GDSC,
  DepMap, NCI60) — GDSC is the supported source for this drug response prediction
  module
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: IMOPAC
  role: Web server / source code providing the 'Machine Learning in Predicting Drug
    Response' module and GDSC RDatas
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: randomForest
  role: Random forest classification with default hyperparameters for drug response
    prediction
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: impute
  role: KNN imputation of missing values in the multiomics feature matrix
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: pROC
  role: Computation of per-fold and average AUC for predictive performance
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: glmnet
  role: Ridge and lasso regression models in the ten-algorithm cross-validation pipeline
  repo: https://github.com/wolfgangsk07/IMOPAC
- name: e1071
  role: Support vector machine model in the cross-validation pipeline
  repo: https://github.com/wolfgangsk07/IMOPAC
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.023
    title: 'IMOPAC: A web server for interactive multiomics and pharmacological analyses
      of patient-derived cancer cell lines'
  source_content_hash: sha256:c16f28f4f2be35b3ad33f6d8e9da89c32b012f6664ccdd1efb50cc37125f8bf0
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
  - adabag
  - caret
  - e1071
  - glm
  - glmnet
  - impute
  - kknn
  - nnet
  - pROC
  - randomForest
  - rpart
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# stratified-cross-validation-model-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates the ability of a user-defined multiomics signature (mRNA, DNA methylation, CNV) to predict response to a user-defined drug from GDSC data using ten machine learning algorithms under 5-fold stratified cross-validation, quantified by average AUC via pROC.

## When to use

Use when a user-defined multiomics signature (e.g. the 15-gene hypoxia signature) must be tested for its ability to predict a user-defined drug response (e.g. Nilotinib) from GDSC cell-line data with an AUC-quantified ML evaluation.

## When NOT to use

- Input is already a trained model or predicted-response vector — this skill only builds and cross-validates models from raw GDSC drug sensitivity plus multiomics features
- The dataset is not one of the IMOPAC-integrated pharmacogenomic databases (GDSC, DepMap, NCI60) — GDSC is the supported source for this drug response prediction module

## Inputs

- GDSC drug sensitivity values for the user-defined drug (e.g. Nilotinib) from IMOPAC GDSC RDatas
- GDSC multiomics feature matrix (mRNA, DNA methylation, CNV) for the user-defined signature genes (e.g. 15-gene hypoxia signature)

## Outputs

- Per-algorithm average AUC table (model_auc_results.csv) with available model coefficients
- Per-fold AUCs and single numeric mean AUC (e.g. auc_nilotinib_hypoxia_rf.csv)
- AUC curve visualization of predictive performance

## How to apply

Load the GDSC drug sensitivity values and multiomics feature matrix (e.g. mRNA expression of the 15 hypoxia-signature genes) from the IMOPAC GDSC RDatas downloaded per urls.txt in the wolfgangsk07/IMOPAC repository. Impute missing feature values with the KNN method from the impute R package (impute 1.66.0), then normalize features with the scale method and binarize each cell line's drug response into sensitive/resistant labels as the dependent variable. Train models — random forest (randomForest 4.6-14), neural networks (nnet), adaboost (adabag 4.2), gradient boosting (caret), SVM (e1071), decision tree (rpart), kNN (kknn), ridge/lasso (glmnet), and logistic regression (glm) — using default hyperparameters under 5-fold stratified cross-validation. Compute the AUC per fold and per algorithm with pROC 1.18.0 and average across folds; write the fold AUCs and mean AUC (e.g. to model_auc_results.csv or auc_nilotinib_hypoxia_rf.csv), including available model coefficients.

## Related tools

- **IMOPAC** (Web server / source code providing the 'Machine Learning in Predicting Drug Response' module and GDSC RDatas) — https://github.com/wolfgangsk07/IMOPAC
- **randomForest** (Random forest classification with default hyperparameters for drug response prediction) — https://github.com/wolfgangsk07/IMOPAC
- **impute** (KNN imputation of missing values in the multiomics feature matrix) — https://github.com/wolfgangsk07/IMOPAC
- **pROC** (Computation of per-fold and average AUC for predictive performance) — https://github.com/wolfgangsk07/IMOPAC
- **glmnet** (Ridge and lasso regression models in the ten-algorithm cross-validation pipeline) — https://github.com/wolfgangsk07/IMOPAC
- **e1071** (Support vector machine model in the cross-validation pipeline) — https://github.com/wolfgangsk07/IMOPAC

## Examples

```
fit <- randomForest(x=feats_scaled, y=as.factor(resp_bin)); a <- auc(pROC::roc(resp_bin, predict(fit, type="prob")[,2]))
```

## Evaluation signals

- Output reports a single numeric average AUC (Area Under Curve) for the drug across the 5 stratified folds, consistent with the metric defined in the IMOPAC 'Machine Learning in Predicting Drug Response' module
- Per-fold AUCs are present for each of the ten algorithms (random forest, nnet, adaboost, caret/gbm, e1071, rpart, kknn, glmnet, glm) before averaging
- No missing ('NA') feature values remain after KNN imputation; features are scaled and responses binarized to sensitive/resistant labels before training
- model_auc_results.csv includes the available model coefficients for each specific model alongside the average AUC

## Limitations

- The drug response predictive model has not yet been validated on an independent dataset — the paper notes 'it is necessary to conduct further validation on an independent dataset in the future analysis'
- The predictive model may be affected by other factors beyond the signature features used
- Drugs with 'NA' sensitivity values in greater than 15% of cell lines must be excluded before applying this skill

## Evidence

- [readme] ML algorithms in README: [evidence span withheld — anchor status: not_found]
