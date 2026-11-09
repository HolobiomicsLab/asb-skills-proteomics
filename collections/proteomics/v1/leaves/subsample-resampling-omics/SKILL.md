---
name: subsample-resampling-omics
description: Use when you need to identify which omic features (e.g. proteomics variables)
  are stably predictive across a dataset, rather than trusting a single model fit,
  and you want signatures with similar predictive performance.
when_to_use_negative:
- When you need an exhaustive enumeration of all feature combinations rather than
  a stability-based shortcut to signatures with similar predictive performance
- When a single deterministic feature-selection step is required (e.g. a fixed filter
  with no resampling), since this skill's importance scores depend on random subsampling
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: muSignAl
  role: R package implementing the subsample-resampling workflow that computes feature
    importance as the proportion of subsample models in which a feature appears
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits the Elastic Net generalized linear model (alpha = 0.9) on each random
    80% subsample
- name: pROC
  role: Evaluates predictive performance of each subsample model by computing the
    AUC used for the AUC > 0.5 filter
- name: R v3.6
  role: Language environment in which muSignAl was coded and the resampling workflow
    is run
- name: devtools
  role: Used to install muSignAl from GitHub via devtools::install_github
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:d8f3b07e03ac869ffdf023e84c7f99b4f8f8a9ee282e874076f1935236d9e559
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# subsample-resampling-omics

## Summary

Resampling-based skill from muSignAl: generate many random subsamples of an omic dataset, fit a predictive model on each, and score feature stability. It identifies features whose repeated selection across models is high, bypassing exhaustive exploration of feature combinations.

## When to use

Use when you need to identify which omic features (e.g. proteomics variables) are stably predictive across a dataset, rather than trusting a single model fit, and you want signatures with similar predictive performance.

## When NOT to use

- When you need an exhaustive enumeration of all feature combinations rather than a stability-based shortcut to signatures with similar predictive performance
- When a single deterministic feature-selection step is required (e.g. a fixed filter with no resampling), since this skill's importance scores depend on random subsampling

## Inputs

- Sample omic dataset in xlsx format (e.g. AD.xlsx from the muSignAl example folder), with samples as rows and features as columns
- Resampling parameters: number of subsamples (100), subsample fraction (80% of samples)
- Model parameters: glmnet Elastic Net mixing parameter alpha = 0.9, FI threshold k = 0.9

## Outputs

- Set of selected features with feature importance (FI) >= 0.9, written to an output file such as sign_cvd2.xlsx
- Predictive performance (AUC from pROC) for each retained subsample model (all must exceed 0.5)

## How to apply

Generate 100 random subsamples, each holding 80% of the samples from the input omic table (e.g. the AD.xlsx sample dataset). On each subsample, fit an Elastic Net generalized linear model with glmnet using alpha = 0.9 (90% LASSO / 10% Ridge penalty mix) and evaluate predictive performance with pROC to obtain the AUC; discard models whose AUC is not greater than 0.5. For each candidate feature, compute feature importance (FI) as the proportion of retained models in which that feature appears, and keep features with FI >= k = 0.9 (default threshold). The rationale is that features recurring across many independently fitted subsample models are robust, so the stability proportion replaces an exhaustive search over feature combinations. Selected features are exported as a signature (e.g. sign_cvd2.xlsx), and the resampling can be repeated to obtain multiple signatures with similar predictive performance.

## Related tools

- **muSignAl** (R package implementing the subsample-resampling workflow that computes feature importance as the proportion of subsample models in which a feature appears) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits the Elastic Net generalized linear model (alpha = 0.9) on each random 80% subsample)
- **pROC** (Evaluates predictive performance of each subsample model by computing the AUC used for the AUC > 0.5 filter)
- **R v3.6** (Language environment in which muSignAl was coded and the resampling workflow is run)
- **devtools** (Used to install muSignAl from GitHub via devtools::install_github)

## Examples

```
install.packages(c("glmnet","pROC")); devtools::install_github("ShuklaLab/muSignAl")
```

## Evaluation signals

- Every retained subsample model must have AUC > 0.5 as computed with pROC
- Every exported feature in the final output must have feature importance FI >= k = 0.9 (i.e. it appears in at least 90% of retained models)
- The final output file (e.g. sign_cvd2.xlsx) is produced, matching the structure of the muSignAl example output
- Exactly 100 random subsamples, each holding 80% of samples, were generated and fitted with glmnet at alpha = 0.9
- Running the package's test script test.R on AD.xlsx reproduces the example output in the package's example folder

## Limitations

- Results depend on random subsampling, so selected signatures may vary between runs; the algorithm instead targets multiple signatures with similar predictive performance
- Subsample models with AUC not greater than 0.5 are discarded, so the FI denominator excludes poorly predictive fits
- The default FI threshold k = 0.9 is stringent and may return an empty feature set on small or noisy datasets
- Requires R v3.6 with glmnet and pROC pre-installed; muSignAl is provided under the MIT license

## Evidence

- [other] Feature importance is the proportion of models in which a feature appears, and a default threshold k = 0.9 applied to this FI is used to select the first set of features: [evidence span withheld — anchor status: not_found]
- [abstract] muSignAl selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features: "which selects multiple signatures with similar predictive performance while systematically bypassing the requirement of exploring all the combinations of features"
- [methods] muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [readme] A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder."
