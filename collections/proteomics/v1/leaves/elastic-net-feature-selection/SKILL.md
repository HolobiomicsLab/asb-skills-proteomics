---
name: elastic-net-feature-selection
description: Use when a tabular omics matrix (e.g. Olink proteomics samples x features
  in .xlsx) must be reduced to predictive biomarker signatures via Elastic Net, with
  FI >= 0.9 cutoff and pROC AUC-based model filtering.
when_to_use_negative:
- When the goal is a single best-performing signature rather than multiple signatures
  with similar predictive performance
- When the input is not a samples x features omics matrix with a classifiable outcome
  (e.g. already-selected features or unstructured data)
- When exhaustive enumeration of all feature combinations is actually required rather
  than the FI-threshold shortcut
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: muSignAl
  role: R package implementing the multiple-signature algorithm whose Elastic Net
    feature-importance selection stage (alpha = 0.9, FI threshold k = 0.9) this skill
    reproduces
  repo: https://github.com/ShuklaLab/muSignAl
- name: glmnet
  role: Fits the Elastic Net generalized linear models (alpha = 0.9) on each 80% subsample
    to rank features by appearance across retained models
- name: pROC
  role: Computes the AUC used to evaluate each fitted subsample model and each enumerated
    signature, enabling the AUC > 0.5 model filter
- name: devtools
  role: Used to install muSignAl from GitHub via devtools::install_github("ShuklaLab/muSignAl")
  repo: https://github.com/ShuklaLab/muSignAl
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202200252
    title: 'muSignAl: An algorithm to search for multiple omic signatures with similar
      predictive performance'
  source_content_hash: sha256:ccb1b789e6e1c05554c8a928f71fe39c7c119e4b73f8f89c5d89e95cd0357bac
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# elastic-net-feature-selection

## Summary

Feature-importance-driven Elastic Net selection stage of the muSignAl algorithm: repeated glmnet model fitting (alpha = 0.9, 90% LASSO / 10% Ridge) over 100 random 80% subsamples defines feature importance (FI) as the proportion of retained models in which a feature appears, enabling enumeration of multiple omic signatures with similar predictive performance without exploring all feature combinations.

## When to use

Use when a tabular omics matrix (e.g. Olink proteomics samples x features in .xlsx) must be reduced to predictive biomarker signatures via Elastic Net, with FI >= 0.9 cutoff and pROC AUC-based model filtering.

## When NOT to use

- When the goal is a single best-performing signature rather than multiple signatures with similar predictive performance
- When the input is not a samples x features omics matrix with a classifiable outcome (e.g. already-selected features or unstructured data)
- When exhaustive enumeration of all feature combinations is actually required rather than the FI-threshold shortcut

## Inputs

- Tabular omics feature matrix in .xlsx format (e.g. AD.xlsx: 77 patient samples x 91 protein features from Olink Proteomics CVD-II panel) with a binary outcome
- FI threshold parameter k (default 0.9)
- Elastic Net mixing parameter alpha (0.9)

## Outputs

- Selected feature/signature sets with every feature satisfying FI >= 0.9
- Per-signature predictive performance (pROC AUC values) exported as sign_cvd2.xlsx

## How to apply

Load the input feature table (e.g. AD.xlsx, 77 samples x 91 protein features) into R v3.6 with glmnet and pROC pre-installed. Generate 100 random subsamples holding 80% of samples; on each, fit an Elastic Net generalized linear model with glmnet using alpha = 0.9 (90% LASSO / 10% Ridge mix) and evaluate predictive performance with pROC to obtain the AUC. Discard any fitted model whose AUC is not greater than 0.5, since such models are no better than chance. For each candidate feature, compute FI as the proportion of retained models in which the feature appears, and keep features with FI >= k = 0.9 to form the first selected feature set whose predictive performance is then calculated. This FI cutoff enumerates multiple candidate signatures without exploring all feature combinations, and results (e.g. sign_cvd2.xlsx) carry per-signature AUC values.

## Related tools

- **muSignAl** (R package implementing the multiple-signature algorithm whose Elastic Net feature-importance selection stage (alpha = 0.9, FI threshold k = 0.9) this skill reproduces) — https://github.com/ShuklaLab/muSignAl
- **glmnet** (Fits the Elastic Net generalized linear models (alpha = 0.9) on each 80% subsample to rank features by appearance across retained models)
- **pROC** (Computes the AUC used to evaluate each fitted subsample model and each enumerated signature, enabling the AUC > 0.5 model filter)
- **devtools** (Used to install muSignAl from GitHub via devtools::install_github("ShuklaLab/muSignAl")) — https://github.com/ShuklaLab/muSignAl

## Examples

```
install.packages(c("glmnet","pROC")); install.packages("devtools"); devtools::install_github("ShuklaLab/muSignAl")
```

## Evaluation signals

- Every retained subsample model must have pROC AUC > 0.5; models with AUC <= 0.5 are discarded
- Every exported feature must satisfy FI >= k = 0.9 (proportion of retained models in which the feature appears)
- Output file sign_cvd2.xlsx is produced and matches the reference output in the package example folder
- The run uses 100 random 80% subsamples with glmnet alpha = 0.9 (90% LASSO / 10% Ridge penalty mix) under R v3.6
- On the Brunner et al. reference dataset (77 samples, 91 features, k = 0.9), the full pipeline yields 1984 signatures of which 158 are unique and 47 have AUC > 0.95

## Limitations

- Requires R v3.6 with glmnet and pROC pre-installed; the package is updated by re-running the same devtools::install_github command
- The AUC > 0.5 filter can discard many subsample models, so FI is defined only over retained models
- The FI threshold k = 0.9 is a default that controls how many candidate signatures are enumerated; different k values change the selected feature set

## Evidence

- [abstract] we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance: "we have developed an algorithm muSignAl (multiple signature algorithm) which selects multiple signatures with similar predictive performance"
- [methods] R v3.6 with glmnet and pROC: "muSignAl was coded in R v3.6, and require a couple of packages (`glmnet` and `pROC`) to be pre-installed"
- [methods] example folder assets: "A test script (`test.R`), ran on a sample dataset (`AD.xlsx`) and its final output (`sign_cvd2.xlsx`) are present in the `example` folder"
- [readme] README purpose statement: "An R package for detecting multiple signatures with similar predictive performance."
- [readme] README installation: "install.packages(c("glmnet","pROC"))"
