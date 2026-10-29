---
name: boosted-tree-classifier-training
description: Use when a set of extracted/embedded feature vectors must be fed to boosted-tree
  or ensemble classifiers and compared (e.g. CatBoost vs XGBoost vs RF vs GBDT) with
  CV-based metrics such as Acc, Sp, Sn, MCC and AUC.
when_to_use_negative:
- Input is already a trained model output or prediction — this skill trains classifiers
  from feature vectors, not from prior predictions
- The task is feature extraction itself (CBOW/word-embedding/BOW generation via GetCBOWFeatures.py),
  not downstream classifier training
- Only a single classifier is required with no comparative benchmark against other
  ensemble algorithms
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: XGBoost
  role: Final classifier trained on the intercepted Layer-1 deep-learning features;
    achieved the highest Acc, MCC and AUC in the 5-fold × 20-times cross-validation
    comparison
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: CatBoost
  role: Default algorithm used to build the classifiers during the comparison; noted
    as much faster to train than the other mentioned algorithms
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: Extreme Gradient Boosting
  role: Receives the obtained (Layer-1) features as the terminal classification step
    of the EMCBOW-GPCR pipeline
  repo: https://github.com/454170054/EMCBOW-GPCR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:25db367310f256356971ef8dc88d04fc583349f421f65d94e16a9e05db97107c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# boosted-tree-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and compare gradient-boosted tree classifiers (XGBoost, CatBoost, GBDT, random forest) on feature vectors produced by an upstream pipeline — here, Layer-1 output features of a deep-learning model in the EMCBOW-GPCR GPCR-identification workflow — selecting the best algorithm via repeated cross-validation.

## When to use

Use when a set of extracted/embedded feature vectors must be fed to boosted-tree or ensemble classifiers and compared (e.g. CatBoost vs XGBoost vs RF vs GBDT) with CV-based metrics such as Acc, Sp, Sn, MCC and AUC.

## When NOT to use

- Input is already a trained model output or prediction — this skill trains classifiers from feature vectors, not from prior predictions
- The task is feature extraction itself (CBOW/word-embedding/BOW generation via GetCBOWFeatures.py), not downstream classifier training
- Only a single classifier is required with no comparative benchmark against other ensemble algorithms

## Inputs

- Layer-1 (deep-learning intermediate) feature vectors from the EMCBOW-GPCR pipeline
- GPCR benchmark dataset labels (GPCR vs non-GPCR) in FASTA-derived form

## Outputs

- Trained classifier model (XGBoost selected as final algorithm)
- Classifier comparison table (classifier_comparison_metrics.csv) with Acc, Sp, Sn, MCC, AUC as mean ± std over 100 CV runs

## How to apply

Intercept the intermediate representation of the upstream pipeline as input features — in EMCBOW-GPCR the output of Layer 1 of the deep-learning model is intercepted as the feature vectors. Train the candidate classifiers (CatBoost, a deep-learning model, GBDT, random forest, and XGBoost/Extreme Gradient Boosting) on these features; CatBoost is the paper's default algorithm, while XGBoost is selected as the final classifier. Evaluate every classifier with 5-fold cross-validation repeated 20 times on the GPCR benchmark dataset (100 CV runs total) and compute Accuracy, Specificity, Sensitivity, Matthews correlation coefficient (MCC) and AUC, reported as mean ± standard deviation. Select the final algorithm by comparing these metrics; in the source study XGBoost attained the highest Acc, MCC and AUC (Acc 92.91±0.10, AUC 95.56±0.13).

## Related tools

- **XGBoost** (Final classifier trained on the intercepted Layer-1 deep-learning features; achieved the highest Acc, MCC and AUC in the 5-fold × 20-times cross-validation comparison) — https://github.com/454170054/EMCBOW-GPCR
- **CatBoost** (Default algorithm used to build the classifiers during the comparison; noted as much faster to train than the other mentioned algorithms) — https://github.com/454170054/EMCBOW-GPCR
- **Extreme Gradient Boosting** (Receives the obtained (Layer-1) features as the terminal classification step of the EMCBOW-GPCR pipeline) — https://github.com/454170054/EMCBOW-GPCR

## Examples

```
python GetResults.py
```

## Evaluation signals

- The reproduced XGBoost row matches the reported metrics: Acc 92.91±0.10, Sp 97.34±0.11, Sn 74.45±0.41, MCC 76.32±0.34, AUC 95.56±0.13
- All five classifiers (CatBoost, DL, GBDT, RF, XGBoost) are evaluated under identical conditions: 5-fold CV repeated 20 times on the same benchmark dataset
- XGBoost attains the highest Acc, MCC and AUC among the tested algorithms, matching the paper's selection rationale
- Every metric is aggregated as mean ± standard deviation over 100 cross-validation runs, not a single split
- A classifier_comparison_metrics.csv table is written containing all five classifier rows

## Limitations

- Version numbers of the software tools (Gensim, CatBoost, XGBoost) are not stated in the paper; they must be taken from the repository's requirements.txt, and the code expects Python 3.7.0 in a fresh virtual environment
- Models must be generated before prediction: running Predict.py requires GetResults.py to have been run first to produce the models
- The upstream DL feature quality is sensitive to hyperparameters (word-embedding N, DL layer/unit structure) that are flexible and hard to optimize, which propagates into classifier performance

## Evidence

- [results] XGBoost selected as final classifier: "the Acc, Mcc and AUC value of XGBoost is the highest. Therefore, XGBoost algorithm is selected as the final algorithm to build classifier in this work"
- [discussion] Layer-1 features as XGBoost input: "we intercepted the output of Layer 1 as the input features of XGBoost which is a fairly powerful, flexible and efficient algorithm"
- [results] CatBoost as default classifier algorithm: "CatBoost algorithm are selected as the default algorithm to build classifiers"
- [readme] Model generation prerequisite in README: "Notice: When running for the first time, please run the python file named GetResults.py to generate models."
