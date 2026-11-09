---
name: feature-importance-frequency-estimation
description: Use when building omic feature-selection pipelines on high-dimensional
  datasets where stability of selected biomarkers across resampled models, not a single
  model's coefficients, should define feature importance.
when_to_use_negative:
- The goal is a single best-performing signature rather than multiple signatures with
  similar predictive performance — muSignAl is designed to find several equally good
  signatures while bypassing exhaustive feature-combination search.
- Only a small number of resampled models is available, so the proportion-of-models
  FI estimate is unstable below the intended 100-subsample regime.
- The dataset is not suited to Elastic Net glmnet modeling (e.g. non-predictive labels),
  since models with AUC <= 0.5 are discarded and few models may survive.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: muSignAl
  role: R package implementing the multiple-signature algorithm whose feature-selection
    stage computes FI as the proportion of models in which a feature appears and filters
    at k=0.9
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits the Elastic Net generalized linear models (alpha=0.9) on each 80% subsample
    from which feature appearance frequencies are counted
- name: pROC
  role: Computes AUC to evaluate each subsample model so that models with AUC <= 0.5
    are discarded before FI is computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:9698437aeedcf83c6cb5917354df4510e189092ac11f152afe3bd1f5f90d6892
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShuklaLab/muSignAl
  tools:
  - R
  - glmnet
  - muSignAl
  - pROC
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# feature-importance-frequency-estimation

## Summary

Estimates feature importance as the proportion of Elastic Net (glmnet, alpha=0.9) subsample models in which a feature appears, then selects features whose FI meets a threshold k=0.9, as used in the muSignAl signature-discovery workflow.

## When to use

Use when building omic feature-selection pipelines on high-dimensional datasets where stability of selected biomarkers across resampled models, not a single model's coefficients, should define feature importance.

## When NOT to use

- The goal is a single best-performing signature rather than multiple signatures with similar predictive performance — muSignAl is designed to find several equally good signatures while bypassing exhaustive feature-combination search.
- Only a small number of resampled models is available, so the proportion-of-models FI estimate is unstable below the intended 100-subsample regime.
- The dataset is not suited to Elastic Net glmnet modeling (e.g. non-predictive labels), since models with AUC <= 0.5 are discarded and few models may survive.

## Inputs

- Sample-by-feature omic dataset in Excel format (e.g. AD.xlsx from the muSignAl example folder), with class labels for AUC evaluation
- FI threshold k (default 0.9)
- Number of subsamples (100) and subsample size (80% of samples)

## Outputs

- First set of selected features, each with FI >= 0.9, exported to an output file such as sign_cvd2.xlsx
- Per-feature FI values (proportion of retained models containing the feature)

## How to apply

Load the input dataset (e.g. the proteomics example AD.xlsx) into R v3.6 and generate 100 random subsamples, each retaining 80% of samples. On each subsample fit an Elastic Net generalized linear model with glmnet using alpha=0.9 (90% LASSO / 10% Ridge penalty mix) and score predictive performance with pROC to obtain the AUC. Discard fitted models whose AUC is not greater than 0.5, since such models perform no better than chance. For each candidate feature compute feature importance (FI) as the proportion of retained models in which that feature appears, and keep features with FI >= k=0.9. The selected features form the first signature whose predictive performance is then calculated and written to the output file (e.g. sign_cvd2.xlsx); the frequency-based definition deliberately bypasses exploring all feature combinations.

## Related tools

- **muSignAl** (R package implementing the multiple-signature algorithm whose feature-selection stage computes FI as the proportion of models in which a feature appears and filters at k=0.9) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits the Elastic Net generalized linear models (alpha=0.9) on each 80% subsample from which feature appearance frequencies are counted)
- **pROC** (Computes AUC to evaluate each subsample model so that models with AUC <= 0.5 are discarded before FI is computed)

## Examples

```
install.packages(c("glmnet","pROC")); devtools::install_github("ShuklaLab/muSignAl"); Rscript test.R
```

## Evaluation signals

- Every model retained for FI computation has AUC strictly greater than 0.5 as measured by pROC.
- Every exported feature in the final output (sign_cvd2.xlsx) has FI >= 0.9.
- FI values are proportions in [0,1] computed over the retained models (100 subsamples of 80% of samples, glmnet alpha=0.9).
- Running the packaged test.R on AD.xlsx reproduces the expected signature output sign_cvd2.xlsx.
- The pipeline requires R v3.6 with glmnet and pROC pre-installed, matching the documented environment.

## Limitations

- FI is a frequency-based stability measure over Elastic Net subsample models, not a coefficient magnitude, so features consistently selected by weak models are treated the same as those from strong ones — models with AUC <= 0.5 must be discarded first.
- The demonstrated workflow is on a proteomics dataset (AD.xlsx with output sign_cvd2.xlsx); behavior on other omic modalities is not shown in the provided context.
- muSignAl was coded in R v3.6, so deviations in R version or missing glmnet/pROC dependencies are untested in the provided context.

## Evidence

- [methods] FI as proportion of models and threshold k: [evidence span withheld — anchor status: not_found]
- [methods] glmnet alpha and pROC AUC filtering: [evidence span withheld — anchor status: not_found]
- [readme] R environment and dependencies: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [readme] Example inputs and outputs: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder."
- [abstract] Purpose of the algorithm: "we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance"
