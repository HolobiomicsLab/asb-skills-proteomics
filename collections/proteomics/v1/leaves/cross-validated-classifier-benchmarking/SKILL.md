---
name: cross-validated-classifier-benchmarking
description: Use when a small labeled training set (e.g., 364 sequences) must be split
  across multiple classifier and feature-selection methods to identify the -performing
  combination before independent testing.
when_to_use_negative:
- Input is already a single fixed model configuration — use direct cross-validation
  of that model rather than a full classifier × feature-selection grid
- The evaluation goal is generalization on truly unseen data — an independent test
  set (as in prPred-DRLF, 92 samples) should supplement or replace training-set CV
  comparisons
- Feature vectors are not yet computed from raw sequences — embedding via BiLSTM/UniRep/TAPE-BERT
  must precede this benchmarking
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0623
tools:
- name: prPred-DRLF
  role: Source toolkit providing the dataset, feature vectors, and cross-validation
    benchmarking pipeline
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Both a feature-selection method and one of the four classifiers benchmarked;
    LGBM + LGBM was the best combination
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: MRMD3.0
  role: Feature-selection method (max-relevance-max-distance) compared against RF
    and LGBM ranking
- name: Python 3.7
  role: Runtime environment for the prPred-DRLF benchmarking scripts (conda env python=3.7)
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:8c53de0f82169876943be3e44736b2eb98a0de64eb86de796b621ca798adb16c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - LightGBM (LGBM)
  - MRMD3.0
  - Python
  - Python 3.7
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-validated-classifier-benchmarking

## Summary

Benchmark classifier × feature-selection combinations (SVM, LGBM, RF, ET × RF, LGBM, MRMD3.0) using fivefold cross-validation on a training set, evaluating precision, recall, ACC, F1-score, and AUC to select the best configuration.

## When to use

Use when a small labeled training set (e.g., 364 sequences) must be split across multiple classifier and feature-selection methods to identify the best-performing combination before independent testing.

## When NOT to use

- Input is already a single fixed model configuration — use direct cross-validation of that model rather than a full classifier × feature-selection grid
- The evaluation goal is generalization on truly unseen data — an independent test set (as in prPred-DRLF, 92 samples) should supplement or replace training-set CV comparisons
- Feature vectors are not yet computed from raw sequences — embedding via BiLSTM/UniRep/TAPE-BERT must precede this benchmarking

## Inputs

- Labeled training set (364 plant R protein sequences: 152 positive, 304 negative before split, labels 1/0)
- Fused BiLSTM + UniRep 5505D feature matrix for the training set
- Feature-selection method choices: RF, LGBM, MRMD3.0
- Classifier choices: SVM, LGBM, RF, ET

## Outputs

- Cross-validation metrics table (precision, recall, ACC, F1-score, AUC per feature-selection × classifier combination, e.g., cv_benchmark_results.csv)
- Identified best configuration (LGBM feature selection + LGBM classifier)

## How to apply

Load the labeled training set and its feature matrix (here the fused BiLSTM 3605D + UniRep 1900D = 5505D vectors for 364 training sequences from the 456-sequence CD-HIT-filtered plant R protein dataset). Rank features and select subsets with each feature-selection method (RF, LGBM, MRMD3.0) separately. Train each classifier (SVM, LGBM, RF, extra tree) on every selected subset using fivefold cross-validation, computing precision, recall, accuracy (ACC), F1-score, and AUC (ROC of true positive rate vs false positive rate) for every combination. Tabulate all metrics (e.g., to cv_benchmark_results.csv) and compare to identify the best configuration; in prPred-DRLF this was LGBM feature selection + LGBM classifier (ACC 0.970, F1 0.953, AUC 0.988).

## Related tools

- **prPred-DRLF** (Source toolkit providing the dataset, feature vectors, and cross-validation benchmarking pipeline) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Both a feature-selection method and one of the four classifiers benchmarked; LGBM + LGBM was the best combination) — https://github.com/Wangys-prog/prPred-DRLF
- **MRMD3.0** (Feature-selection method (max-relevance-max-distance) compared against RF and LGBM ranking)
- **Python 3.7** (Runtime environment for the prPred-DRLF benchmarking scripts (conda env python=3.7)) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
conda create -n prPred-DRLF python=3.7 && conda activate prPred-DRLF && pip3 install -r requirements.txt
```

## Evaluation signals

- Best configuration reproduces the reported LGBM + LGBM cross-validated performance: ACC 0.970, F1-score 0.953, AUC 0.988
- Metrics table contains all 12 feature-selection × classifier combinations with precision, recall, ACC, F1-score, and AUC columns populated
- Fivefold CV is performed on the 364-sequence training set only, with labels 1 (R protein) / 0 (non-R protein)
- AUC is computed from the ROC curve of true positive rate vs false positive rate, not from accuracy alone
- Feature subsets differ per selection method (RF, LGBM, MRMD3.0 rankings), producing distinct metric rows rather than identical results

## Limitations

- Deep representation learning features usually require an elevated level of computing resources (stated limitation); GPU recommended per README
- Parallel computing is not yet incorporated into the program; authors plan this in future studies for computational efficiency
- Benchmarking is on training-set cross-validation only; independent test evaluation (92 samples) is a separate step

## Evidence

- [intro] Three feature selection techniques, including random forest (RF), LGBM, and max-relevance-max-distance (MRMD), were applied to select an optimal feature subset from the fused BiLSTM + UniRep feature: [evidence span withheld — anchor status: not_found]
- [intro] then the feature subset was used as input to four classifiers, including support vector machine (SVM), LGBM, RF, and extra tree (ET).: [evidence span withheld — anchor status: not_found]
- [abstract] The results show that the fused features of bidirectional long short-term memory (BiLSTM) embedding and unified representation (UniRep) embedding have a better performance than other features: [evidence span withheld — anchor status: not_found]
