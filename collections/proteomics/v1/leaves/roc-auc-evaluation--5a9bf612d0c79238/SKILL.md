---
name: roc-auc-evaluation
description: Use when comparing binary R-protein classifiers (prPred-DRLF, prPred,
  HMMER) or feature/classifier combinations on the 364-sample training or 92-sample
  independent test sets, where TP/FP/TN/FN metrics and ROC analysis are needed.
when_to_use_negative:
- Negative training set sequences need to be scored with HMMER — most are annotated
  incompletely, so HMMER evaluation must be restricted to the positive training set
  only
- Inputs are raw sequences that have not yet been encoded by BiLSTM/UniRep/TAPE-BERT
  or classified — metric computation presupposes predicted labels/scores
- The task is multi-class or non-binary R-protein classification; the TP/FP/TN/FN
  and ROC definitions here apply only to the binary R vs non-R protein problem
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: prPred-DRLF
  role: Model under evaluation; its predictions on the independent test set supply
    TP/FP/TN/FN counts for metric computation
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred
  role: Baseline predictor whose metrics are compared against prPred-DRLF deltas (ACC
    +2.20%, F1 +4.29%, AUC +4.91%)
- name: HMMER
  role: Profile-alignment baseline evaluated on the positive training set only, with
    recall compared against prPred-DRLF
- name: LightGBM (LGBM)
  role: Classifier whose cross-validated and independent-test outputs (ACC 0.970/F1
    0.953 CV; ACC 0.956/F1 0.933/AUC 0.997 test) are scored with these metrics
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:6aff463c0407ee6f58959799482411db74ffbdc177ac211694333ef5588dcdc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - HMMER
  - LightGBM (LGBM)
  - MRMD3.0
  - Python
  - eFeature
  - prPred
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# roc-auc-evaluation

## Summary

Evaluate plant R protein predictors using precision, recall, accuracy (ACC), F1-score, and the area under the ROC curve (AUC), as done for prPred-DRLF versus the prPred and HMMER baselines on the Wang et al. [9] plant R protein dataset.

## When to use

Use when comparing binary R-protein classifiers (prPred-DRLF, prPred, HMMER) or feature/classifier combinations on the 364-sample training or 92-sample independent test sets, where TP/FP/TN/FN metrics and ROC analysis are needed.

## When NOT to use

- Negative training set sequences need to be scored with HMMER — most are annotated incompletely, so HMMER evaluation must be restricted to the positive training set only
- Inputs are raw sequences that have not yet been encoded by BiLSTM/UniRep/TAPE-BERT or classified — metric computation presupposes predicted labels/scores
- The task is multi-class or non-binary R-protein classification; the TP/FP/TN/FN and ROC definitions here apply only to the binary R vs non-R protein problem

## Inputs

- Predicted R/non-R protein labels or scores from prPred-DRLF, prPred, or HMMER runs on the 92-sample independent test set
- Ground-truth labels (1 = R protein, 0 = non-R protein) for the 364 training and 92 independent test sequences from the Wang et al. [9] dataset (https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset)
- HMMER profile alignment scores on the 152 positive training R proteins

## Outputs

- Per-model metrics table: precision, recall, accuracy (ACC), F1-score, and AUC for prPred-DRLF, prPred, and HMMER
- ROC curve (true positive rate vs false positive rate) and computed AUC value
- Performance deltas of prPred-DRLF over prPred (ACC, F1-score, AUC), e.g. cv_benchmark_results.csv or feature_combination_metrics.csv

## How to apply

Split the CD-HIT-filtered Wang et al. [9] dataset (456 sequences: 152 positive, 304 negative; labels 1 = R protein, 0 = non-R protein) into 364 training and 92 independent test samples. For each model run, compute precision = TP/(TP+FP), recall = TP/(TP+FN), accuracy = (TP+TN)/(TP+FP+TN+FN), F1-score, and AUC from the true-positive-rate vs false-positive-rate ROC curve. Use fivefold cross-validation on the training data when examining feature-selection or classifier combinations, and the 92-sample independent test set for final comparison. When benchmarking against HMMER, score only the positive training set with the profile alignments because most negative training set sequences are annotated incompletely under HMMER. Judge success by matching reported values: prPred-DRLF improved over prPred by ACC +2.20%, F1 +4.29%, AUC +4.91%; BiLSTM+UniRep with LGBM reached ACC 0.970 and F1 0.953 under 5-fold CV and accuracy 0.956, F1 0.933, AUC 0.997 on the independent test.

## Related tools

- **prPred-DRLF** (Model under evaluation; its predictions on the independent test set supply TP/FP/TN/FN counts for metric computation) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred** (Baseline predictor whose metrics are compared against prPred-DRLF deltas (ACC +2.20%, F1 +4.29%, AUC +4.91%))
- **HMMER** (Profile-alignment baseline evaluated on the positive training set only, with recall compared against prPred-DRLF)
- **LightGBM (LGBM)** (Classifier whose cross-validated and independent-test outputs (ACC 0.970/F1 0.953 CV; ACC 0.956/F1 0.933/AUC 0.997 test) are scored with these metrics) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Independent-test prPred-DRLF deltas over prPred must match ACC +2.20%, F1-score +4.29%, AUC +4.91%
- Best CV configuration (LGBM feature selection + LGBM classifier) must reproduce ACC 0.970 and F1 0.953 (AUC 0.988) on the 364-sample training set
- Independent-test BiLSTM+UniRep row must match accuracy 0.956, precision 0.967, recall 0.905, F1 0.933, AUC 0.997 within tolerance
- prPred-DRLF recall must exceed HMMER recall, consistent with the reported overall improvement in accuracy, F1-score, AUC, and recall
- All seven BiLSTM/UniRep/TAPE-BERT feature combinations (including fused BiLSTM+UniRep = 5505D) must yield complete metric rows

## Limitations

- HMMER cannot be evaluated on the full negative training set because most negative training set sequences are annotated incompletely under HMMER
- Deep representation learning feature methods (BiLSTM, UniRep, TAPE-BERT) require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program, limiting computational efficiency
- No accession or source for the independent test dataset is stated beyond the Wang et al. [9] dataset link

## Evidence

- [abstract] The model was evaluated using an independent test achieving an accuracy of 0.956, F1-scoreof0.933,andareaunderthereceiveroperatingcharacteristic(ROC)curve(AUC)of 0.997.: [evidence span withheld — anchor status: not_found]
- [results] On the independent test dataset, prPred-DRLF outperformed prPred, increasing the ACC by 2.20%, F1-score by 4.29%, and AUC by 4.91%.: [evidence span withheld — anchor status: not_found]
