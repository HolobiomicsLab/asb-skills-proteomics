---
name: classification-metrics-computation
description: Use when two or more binary protein classifiers (e.g., prPred-DRLF vs
  prPred vs HMMER) must be compared on the same 92-sequence independent test set using
  ACC, F1-score, AUC, and recall deltas.
when_to_use_negative:
- The task is feature extraction or model training (e.g., CD-HIT filtering at 30%
  identity, BiLSTM/UniRep embedding fusion) rather than evaluating and comparing already-trained
  predictors
- Only a single classifier's predictions exist with no baseline (prPred or HMMER)
  to compare against, so no delta analysis is possible
- The negative training set must be scored with HMMER, since most negative sequences
  are annotated incompletely and HMMER evaluation is restricted to the positive training
  set
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: prPred-DRLF
  role: Primary predictor under evaluation; its R/non-R predictions on the 92-sequence
    independent test set feed the TP/FP/TN/FN metric computation
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred
  role: State-of-the-art baseline predictor; evaluated on the same independent test
    set to compute ACC/F1/AUC deltas relative to prPred-DRLF
- name: HMMER
  role: Profile alignment baseline scored on the positive training set only, providing
    the recall comparison against prPred-DRLF
- name: Python 3.7
  role: Environment for running prPred-DRLF and the metric/ROC analysis scripts
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:b3562ceeeb2d942966a615666c683d87f25802ffeda1657e450572c8692ba539
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - HMMER
  - Python
  - Python 3.7
  - prPred
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# classification-metrics-computation

## Summary

Computes precision, recall, accuracy (ACC), F1-score, and AUC from TP/FP/TN/FN counts and ROC analysis to benchmark plant R protein predictors (prPred-DRLF, prPred, HMMER) on a shared independent test set.

## When to use

Use when two or more binary protein classifiers (e.g., prPred-DRLF vs prPred vs HMMER) must be compared on the same 92-sequence independent test set using ACC, F1-score, AUC, and recall deltas.

## When NOT to use

- The task is feature extraction or model training (e.g., CD-HIT filtering at 30% identity, BiLSTM/UniRep embedding fusion) rather than evaluating and comparing already-trained predictors
- Only a single classifier's predictions exist with no baseline (prPred or HMMER) to compare against, so no delta analysis is possible
- The negative training set must be scored with HMMER, since most negative sequences are annotated incompletely and HMMER evaluation is restricted to the positive training set

## Inputs

- Independent test set of 92 protein sequences in FASTA format from the Wang et al. [9] dataset
- Positive training set of 152 R protein sequences (for HMMER profile alignment scoring)
- Predicted labels or scores from each classifier (prPred-DRLF, prPred, HMMER) on the test sequences
- Ground-truth R/non-R labels for the test sequences

## Outputs

- Per-classifier table of precision, recall, ACC, F1-score, and AUC
- ROC curves / AUC values for each predictor
- Performance deltas of prPred-DRLF over prPred (ACC, F1-score, AUC) and recall comparison vs HMMER

## How to apply

Load the 92-sequence independent test set from the Wang et al. [9] dataset and run each predictor (prPred-DRLF, prPred, HMMER) on the same sequences to obtain R/non-R predictions. Derive TP/FP/TN/FN counts using the standard definitions and compute precision, recall, ACC, F1-score, and AUC via ROC analysis. For HMMER, restrict evaluation to the positive training set only, because most negative training set sequences are annotated incompletely under HMMER. Compute the performance deltas of prPred-DRLF over prPred (ACC +2.20%, F1-score +4.29%, AUC +4.91%) and compare recall between prPred-DRLF and HMMER; the skill is applied correctly when reproduced deltas match the reported improvements and prPred-DRLF recall exceeds HMMER recall.

## Related tools

- **prPred-DRLF** (Primary predictor under evaluation; its R/non-R predictions on the 92-sequence independent test set feed the TP/FP/TN/FN metric computation) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred** (State-of-the-art baseline predictor; evaluated on the same independent test set to compute ACC/F1/AUC deltas relative to prPred-DRLF)
- **HMMER** (Profile alignment baseline scored on the positive training set only, providing the recall comparison against prPred-DRLF)
- **Python 3.7** (Environment for running prPred-DRLF and the metric/ROC analysis scripts) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Reproduced deltas must match reported improvements: ACC +2.20%, F1-score +4.29%, AUC +4.91% of prPred-DRLF over prPred
- prPred-DRLF recall must exceed HMMER recall
- prPred-DRLF independent-test metrics must fall near ACC 0.956, F1-score 0.933, AUC 0.997
- Metric counts must follow standard TP/FP/TN/FN definitions over exactly 92 independent test sequences (from 456 sequences split into 364 training / 92 test)
- HMMER scores must be computed only on the 152 positive training sequences, never on the negative training set

## Limitations

- HMMER cannot be fairly evaluated on the negative training set because most negative sequences are annotated incompletely, restricting the comparison to recall on positives
- Deep representation learning feature methods require an elevated level of computing resources (GPU recommended per README)
- Parallel computing is not yet incorporated into prPred-DRLF, limiting computational efficiency
- Article text states no accession/source for the independent test dataset beyond the GitHub dataset directory

## Evidence

- [abstract] Independent test achieved accuracy 0.956, F1-score 0.933, and AUC 0.997: [evidence span withheld — anchor status: not_found]
- [methods] four evaluation measures are used, including precision, recall, accuracy (ACC), and F1-score: "four evaluation measures are used, including precision, recall, accuracy (ACC), and F1-score"
- [results] HMMER evaluation restricted to positive training set only: "Because most protein sequences from the negative training set are annotated incompletely using the HMMER method, we only evaluated whether the sequences from the positive training set"
- [abstract] prPred-DRLF shows overall improvement in accuracy, F1-score, AUC, and recall: "pr Pred-DRLF shows an overall improvement in accuracy, F1-score, AUC, and recall."
