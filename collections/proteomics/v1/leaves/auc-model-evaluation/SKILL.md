---
name: auc-model-evaluation
description: Use when repeated subsample models (e.g. Elastic Net glmnet fits) must
  be scored for predictive performance and filtered on AUC > 0.5 before downstream
  feature selection.
when_to_use_negative:
- When the input is already a validated feature/signature table rather than fitted
  subsample models — the AUC filter only makes sense per fitted model, not per feature.
- When the goal is to select signatures without measuring predictive performance —
  muSignAl's selection is explicitly tied to 'multiple signatures with similar predictive
  performance', so an AUC evaluation stage is mandatory in this workflow.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: pROC
  role: Computes the AUC used to evaluate each Elastic Net subsample model's predictive
    performance; models with AUC not greater than 0.5 are discarded.
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits the Elastic Net generalized linear models (alpha = 0.9) on each 80% subsample
    whose AUC is then evaluated.
  repo: https://github.com/ShuklaLab/muSignAl
- name: muSignAl
  role: R package (coded in R v3.6) that orchestrates subsampling, model fitting,
    AUC evaluation, and downstream feature-importance selection.
  repo: https://github.com/ShuklaLab/muSignAl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:a32075320c0e67cc5718042a85bbb8713f8060d7b2372842ec65b7e07ecd55d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# auc-model-evaluation

## Summary

Evaluates predictive models built within the muSignAl workflow by computing the area under the ROC curve (AUC) with pROC, and retains only models whose AUC exceeds 0.5 so that feature importance is computed from predictive (better-than-random) subsample models.

## When to use

Use when repeated subsample models (e.g. Elastic Net glmnet fits) must be scored for predictive performance and filtered on AUC > 0.5 before downstream feature selection.

## When NOT to use

- When the input is already a validated feature/signature table rather than fitted subsample models — the AUC filter only makes sense per fitted model, not per feature.
- When the goal is to select signatures without measuring predictive performance — muSignAl's selection is explicitly tied to 'multiple signatures with similar predictive performance', so an AUC evaluation stage is mandatory in this workflow.

## Inputs

- Sample dataset in xlsx format (e.g. AD.xlsx from the muSignAl example folder) with samples as rows and candidate features (e.g. proteomics measurements) as columns
- Fitted subsample models: Elastic Net GLMs (glmnet, alpha = 0.9) trained on 100 random 80% subsamples

## Outputs

- AUC value per fitted subsample model, computed with pROC
- Set of retained models filtered to AUC > 0.5, feeding the feature-importance (FI >= 0.9) selection step
- Final output file with selected signatures (e.g. sign_cvd2.xlsx in the example folder)

## How to apply

In the muSignAl workflow, 100 random subsamples each holding 80% of the samples from the input dataset (e.g. the example AD.xlsx) are generated, and an Elastic Net generalized linear model is fitted on each subsample with glmnet using alpha = 0.9 (a 90% LASSO / 10% Ridge penalty mix). Each fitted model's predictive performance is evaluated with pROC to obtain the AUC. Models whose AUC is not greater than 0.5 are discarded, so only models that outperform random guessing on their subsample contribute to downstream feature-importance estimation. This AUC filter is applied before computing feature importance as the proportion of retained models in which each feature appears (thresholded at FI >= k = 0.9), which keeps the selection tied to genuinely predictive signatures. The final outputs, including the selected features written to sign_cvd2.xlsx in the package example, are validated by confirming every retained model has AUC > 0.5.

## Related tools

- **pROC** (Computes the AUC used to evaluate each Elastic Net subsample model's predictive performance; models with AUC not greater than 0.5 are discarded.) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits the Elastic Net generalized linear models (alpha = 0.9) on each 80% subsample whose AUC is then evaluated.) — https://github.com/ShuklaLab/muSignAl
- **muSignAl** (R package (coded in R v3.6) that orchestrates subsampling, model fitting, AUC evaluation, and downstream feature-importance selection.) — https://github.com/ShuklaLab/muSignAl

## Examples

```
install.packages(c("glmnet","pROC")); devtools::install_github("ShuklaLab/muSignAl"); Rscript test.R
```

## Evaluation signals

- Every retained subsample model has AUC > 0.5; any model with AUC <= 0.5 is discarded before feature-importance computation.
- The number of models evaluated equals the number of generated subsamples (100 subsamples at 80% holdout in the example workflow).
- Only features with FI >= 0.9 (i.e. appearing in at least 90% of retained models) appear in the output file (e.g. sign_cvd2.xlsx).
- The example pipeline runs end-to-end via test.R on AD.xlsx and produces sign_cvd2.xlsx in the example folder.
- Required packages (glmnet, pROC) are pre-installed before running, e.g. via install.packages(c("glmnet","pROC")).

## Limitations

- AUC is a subsample-level performance metric; models with AUC <= 0.5 are excluded, so feature importance is computed only over models that beat random guessing, which can reduce the retained-model pool.
- The workflow was demonstrated on a single proteomics dataset (AD.xlsx) as the package example; behavior on other omic data types was not shown in the provided context.
- muSignAl is coded in R v3.6 and requires glmnet and pROC to be pre-installed, so version/dependency constraints apply.

## Evidence

- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [readme] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [abstract] which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
