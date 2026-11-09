---
name: binary-classifier-metric-evaluation
description: Use when a trained binary classifier's predictions on held-out test samples
  (e.g., the 92-sample plant R protein independent test) must be scored with accuracy,
  precision, recall, F1-score, and AUC and compared to published values.
when_to_use_negative:
- The problem is multi-class or regression — the precision/recall/accuracy/F1/AUC
  formulas here assume binary labels (1 = R protein, 0 = non-R protein)
- No ground-truth labels accompany the predictions — these supervised metrics cannot
  be computed from prediction output alone
- The task is to generate embeddings (eFeature/BiLSTM/UniRep/TAPE-BERT) or train the
  LGBM classifier — this skill only scores already-produced predictions
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3307
tools:
- name: prPred-DRLF
  role: Runs the BiLSTM+UniRep+LGBM predictor on the independent-test FASTA and writes
    the prediction CSV from which the metrics are computed
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Classifier whose independent-test predictions are scored with accuracy, precision,
    recall, F1-score, and AUC; also used for feature selection prior to evaluation
- name: eFeature
  role: Generates the BiLSTM (3605D), UniRep (1900D), and TAPE-BERT (768D) embedding
    vectors that supply the feature combinations being evaluated
  repo: http://lab.malab.cn/soft/eFeature/index.html
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:02ec594afba90338a439db114dc4ebb8f42c84f557ea7322fe6e49002976cf44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - CD-HIT
  - LightGBM (LGBM)
  - Python
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# binary-classifier-metric-evaluation

## Summary

Scores binary plant R protein classifiers (e.g., the LGBM classifier on fused BiLSTM 3605D + UniRep 1900D embeddings) with accuracy, precision, recall, F1-score, and ROC-AUC on the 92-sample independent test set, enabling reproduction of the prPred-DRLF benchmark (accuracy 0.956, F1-score 0.933, AUC 0.997).

## When to use

Use when a trained binary classifier's predictions on held-out test samples (e.g., the 92-sample plant R protein independent test) must be scored with accuracy, precision, recall, F1-score, and AUC and compared to published values.

## When NOT to use

- The problem is multi-class or regression — the precision/recall/accuracy/F1/AUC formulas here assume binary labels (1 = R protein, 0 = non-R protein)
- No ground-truth labels accompany the predictions — these supervised metrics cannot be computed from prediction output alone
- The task is to generate embeddings (eFeature/BiLSTM/UniRep/TAPE-BERT) or train the LGBM classifier — this skill only scores already-produced predictions

## Inputs

- Independent-test FASTA file of plant protein sequences (92 samples, drawn from the Wang et al. [9] dataset after CD-HIT 30% identity filtering)
- Ground-truth binary labels for the test samples (1 = R protein, 0 = non-R protein)
- Prediction CSV (e.g., predict_result.csv) output by prPred-DRLF.py or the trained LGBM classifier

## Outputs

- Metric row per feature combination: accuracy, precision, recall, F1-score, and AUC (e.g., BiLSTM+UniRep: 0.956, 0.967, 0.905, 0.933, 0.997)
- Comparison table saved as feature_combination_metrics.csv
- ROC curve and AUC value for each evaluated classifier

## How to apply

Run the trained classifier — the top model fuses BiLSTM (3605D) and UniRep (1900D) embeddings into a 5505D vector reduced by LGBM feature selection — on the 92-sample independent test set by invoking prPred-DRLF.py with the input FASTA and an output CSV path, then join the predictions (predict_result.csv) with ground-truth labels where 1 = R protein and 0 = non-R protein. Derive the confusion matrix and compute precision TP/(TP+FP), recall TP/(TP+FN), accuracy (TP+TN)/(TP+FP+TN+FN), F1-score, and the area under the ROC curve (AUC); fivefold cross-validation on the 364 training samples is applied beforehand to examine each model's effectiveness. Apply the identical metric battery to every feature combination (BiLSTM, UniRep, TAPE-BERT 768D, and their fusions) so rows are directly comparable, and compile them into feature_combination_metrics.csv. Judge correctness by whether the BiLSTM+UniRep row reproduces the reported independent-test values — accuracy 0.956, precision 0.967, recall 0.905, F1-score 0.933, AUC 0.997 — within a small tolerance and all seven feature combinations yield complete metric rows.

## Related tools

- **prPred-DRLF** (Runs the BiLSTM+UniRep+LGBM predictor on the independent-test FASTA and writes the prediction CSV from which the metrics are computed) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Classifier whose independent-test predictions are scored with accuracy, precision, recall, F1-score, and AUC; also used for feature selection prior to evaluation)
- **eFeature** (Generates the BiLSTM (3605D), UniRep (1900D), and TAPE-BERT (768D) embedding vectors that supply the feature combinations being evaluated) — http://lab.malab.cn/soft/eFeature/index.html

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Reproduced BiLSTM+UniRep row matches the reported independent-test values (accuracy 0.956, precision 0.967, recall 0.905, F1-score 0.933, AUC 0.997) within a small tolerance
- Accuracy recomputed from the confusion matrix via (TP+TN)/(TP+FP+TN+FN); with 92 test samples it must fall on a 1/92 grid (~0.01087 increments per sample)
- All seven feature combinations (BiLSTM, UniRep, TAPE-BERT, and their fusions) yield complete metric rows in feature_combination_metrics.csv
- AUC lies in [0,1] and, as reported, exceeds accuracy (0.997 vs 0.956), consistent with a well-ranked ROC curve
- Metric ordering reflects the imbalanced 152-positive/304-negative dataset: precision (0.967) > accuracy (0.956) > recall (0.905)

## Limitations

- The independent test contains only 92 samples (dataset: 456 sequences, 152 positive, 304 negative after CD-HIT 30% identity filtering), so each misclassification shifts accuracy by ~1.1% and metric confidence intervals are wide
- Accuracy alone can mislead under the 1:2 positive:negative class imbalance; precision, recall, F1-score, and AUC must be reported alongside it
- Deep representation learning features 'usually require an elevated level of computing resources', and parallel computing is not yet incorporated into the program

## Evidence

- [abstract] Independent-test metrics reported: accuracy 0.956, F1-score 0.933, AUC 0.997: [evidence span withheld — anchor status: not_found]
- [methods] Four evaluation measures defined: precision, recall, accuracy (ACC), and F1-score: "four evaluation measures are used, including precision, recall, accuracy (ACC), and F1-score"
- [methods] Fivefold cross-validation examines each model's effectiveness before independent testing: "Fivefold cross-validation on the dataset is used to examine the effectiveness of each model"
- [readme] README CLI invocation that produces the prediction output on the test FASTA: "python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result"
- [discussion] Stated limitation: deep representation learning features demand substantial computing resources: "they usually require an elevated level of computing resources"
