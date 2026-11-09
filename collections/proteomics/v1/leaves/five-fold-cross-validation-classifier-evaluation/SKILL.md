---
name: five-fold-cross-validation-classifier-evaluation
description: Use when several candidate classifiers must be compared on the same precomputed
  feature matrix (e.g., Layer-1 DL features) and a defensible algorithm choice is
  needed, as in the EMCBOW-GPCR work.
when_to_use_negative:
- Input is already a trained predictor or a single model's output — this skill compares
  candidate classifiers, not predictions
- The dataset is too small for 5-fold CV repeated 20 times, or evaluation must use
  an independent test set instead of cross-validation
- Only one algorithm is under consideration and no comparison table is needed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: XGBoost
  role: Extreme Gradient Boosting classifier evaluated on Layer-1 features; selected
    as the final algorithm for building the GPCR predictor
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: CatBoost
  role: Default classifier used to build initial comparison classifiers during the
    5-fold CV with 20 repetitions evaluation
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: Extreme Gradient Boosting
  role: Final algorithm into which the obtained features are fed after dimensionality
    reduction by the DL model
  repo: https://github.com/454170054/EMCBOW-GPCR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:313f146e48cc1ca3339b1d72e3754aa8cd4238dff4258d56a91649c95328dd26
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  repo_url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - CatBoost
  - Extreme Gradient Boosting
  - XGBoost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# five-fold-cross-validation-classifier-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple classifiers (e.g., CatBoost, XGBoost, GBDT, RF, DL) on fixed feature vectors using 5-fold cross-validation repeated 20 times, reporting Acc, Sp, Sn, MCC, and AUC as mean ± SD over the 100 CV runs to select the best algorithm.

## When to use

Use when several candidate classifiers must be compared on the same precomputed feature matrix (e.g., Layer-1 DL features) and a defensible algorithm choice is needed, as in the EMCBOW-GPCR work.

## When NOT to use

- Input is already a trained predictor or a single model's output — this skill compares candidate classifiers, not predictions
- The dataset is too small for 5-fold CV repeated 20 times, or evaluation must use an independent test set instead of cross-validation
- Only one algorithm is under consideration and no comparison table is needed

## Inputs

- Feature matrix of GPCR sequences (Layer-1 DL output features, or concatenated CBOW/BOW features)
- Benchmark dataset labels for GPCR identification
- Trained CBOW/DL feature pipeline from the EMCBOW-GPCR repository

## Outputs

- classifier_comparison_metrics.csv with Acc, Sp, Sn, MCC, and AUC (mean ± SD over 100 CV runs) for each classifier
- Selected final classifier (e.g., XGBoost) with its aggregated performance metrics

## How to apply

Train each candidate classifier — CatBoost (the paper's default), XGBoost, GBDT, random forest, and the deep-learning model — on the same input feature vectors (here, the output intercepted from Layer 1 of the DL model trained on concatenated CBOW word-embedding and BOW features of GPCR sequences). Evaluate every classifier with 5-fold cross-validation repeated 20 times on the benchmark dataset, i.e., 100 CV runs per algorithm. For each run compute Accuracy, Specificity, Sensitivity, Matthews correlation coefficient (MCC), and AUC, then aggregate as mean ± standard deviation. Select the final algorithm by comparing aggregated metrics; in this paper XGBoost won because its Acc, MCC, and AUC were the highest (Acc 92.91±0.10, AUC 95.56±0.13), even though CatBoost trains faster. Write the full comparison table (e.g., classifier_comparison_metrics.csv) so each model's metrics are auditable against the reported values.

## Related tools

- **XGBoost** (Extreme Gradient Boosting classifier evaluated on Layer-1 features; selected as the final algorithm for building the GPCR predictor) — https://github.com/454170054/EMCBOW-GPCR
- **CatBoost** (Default classifier used to build initial comparison classifiers during the 5-fold CV with 20 repetitions evaluation) — https://github.com/454170054/EMCBOW-GPCR
- **Extreme Gradient Boosting** (Final algorithm into which the obtained features are fed after dimensionality reduction by the DL model) — https://github.com/454170054/EMCBOW-GPCR

## Examples

```
git clone https://github.com/454170054/EMCBOW-GPCR; conda create -n emcbow python=3.7.0; pip install -r EMCBOW-GPCR/requirements.txt; python EMCBOW-GPCR/GetResults.py
```

## Evaluation signals

- Reproduced XGBoost row matches the reported Acc 92.91±0.10 and AUC 95.56±0.13 on Layer-1 DL features
- All five classifiers are evaluated over exactly 100 CV runs (5-fold × 20 repetitions) with mean ± SD reported for all five metrics
- XGBoost attains the highest Acc, MCC, and AUC among CatBoost, RF, GBDT, DL, and XGBoost in the comparison table
- classifier_comparison_metrics.csv is written and contains a row per candidate classifier with all five metrics
- Combined-feature baseline is consistent with the paper: combined features AUC 0.9423 > CBOW 0.9391 > BOW 0.9344 under CatBoost

## Limitations

- Version numbers of the software packages (Gensim, CatBoost, XGBoost) are not stated in the paper; consult requirements.txt in the repository
- The hyperparameter N in word embedding is flexible, hard to determine, and may significantly influence feature quality and hence classifier comparison
- The DL model structure (number of layers and units per layer) is flexible and difficult to optimize, affecting the Layer-1 features fed to the classifiers
- Requires Python 3.7.0 virtual environment with packages from requirements.txt; first run must execute GetResults.py to generate models

## Evidence
