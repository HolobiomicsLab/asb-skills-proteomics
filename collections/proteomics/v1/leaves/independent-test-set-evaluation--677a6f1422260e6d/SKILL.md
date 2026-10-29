---
name: independent-test-set-evaluation
description: Use when a trained plant R protein predictor must be fairly benchmarked
  against prior predictors (prPred, HMMER) on a held-out, non-redundant independent
  test set rather than on cross-validation alone.
when_to_use_negative:
- You only need cross-validation performance during feature-selection or classifier
  comparison (SVM/LGBM/RF/ET); use the fivefold CV protocol on the 364 training samples
  instead
- HMMER must be evaluated on negative training sequences — they are annotated incompletely,
  so HMMER scoring is restricted to the positive training set only
- Your dataset has not been deduplicated with CD-HIT at the 30% identity cutoff; redundancy
  would leak homologs between training and test and inflate independent-test metrics
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: prPred-DRLF
  role: Primary predictor evaluated on the independent test set; encodes sequences
    as deep representation learning features (BiLSTM + UniRep fused) and classifies
    R vs non-R with LGBM
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred
  role: State-of-the-art baseline predictor run on the same independent test sequences
    to compute ACC/F1/AUC improvement deltas
- name: HMMER
  role: Profile alignment baseline scored on the positive training set only (negative
    training sequences are annotated incompletely under HMMER)
- name: LightGBM (LGBM)
  role: Classifier producing the final R/non-R predictions evaluated on the independent
    test set
- name: Python 3.7
  role: 'Runtime environment for prPred-DRLF (requirements: joblib 1.0.1, tape_proteins
    0.5, numpy 1.19.2, pandas 1.2.0, Bio 0.4.1, lightgbm 2.3.0)'
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:faa75a6f1bf321231549a2bdecd9136c0caa83b716cb74fc34c6dd021c02e5cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - HMMER
  - LightGBM (LGBM)
  - Python
  - Python 3.7
  - prPred
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# independent-test-set-evaluation

## Summary

Evaluate the prPred-DRLF plant R protein predictor on the held-out 92-sequence independent test set and benchmark it against the prPred and HMMER baselines using precision, recall, ACC, F1-score, and AUC. This validates that reported deltas (ACC +2.20%, F1 +4.29%, AUC +4.91%) reproduce on identical data.

## When to use

Use when a trained plant R protein predictor must be fairly benchmarked against prior predictors (prPred, HMMER) on a held-out, non-redundant independent test set rather than on cross-validation alone.

## When NOT to use

- You only need cross-validation performance during feature-selection or classifier comparison (SVM/LGBM/RF/ET); use the fivefold CV protocol on the 364 training samples instead
- HMMER must be evaluated on negative training sequences — they are annotated incompletely, so HMMER scoring is restricted to the positive training set only
- Your dataset has not been deduplicated with CD-HIT at the 30% identity cutoff; redundancy would leak homologs between training and test and inflate independent-test metrics

## Inputs

- Independent test set FASTA (92 sequences) from the Wang et al. [9] dataset
- Positive training set (152 R proteins) for HMMER profile scoring
- Trained prPred-DRLF model (Python 3.7 toolkit, LGBM classifier)
- prPred predictor predictions on the same test sequences

## Outputs

- Per-method metrics table: precision, recall, accuracy, F1-score, AUC for prPred-DRLF, prPred, and HMMER
- Performance deltas of prPred-DRLF over prPred (ACC +2.20%, F1-score +4.29%, AUC +4.91%)
- ROC curve / AUC analysis for the independent test predictions
- CSV prediction output from prPred-DRLF

## How to apply

Load the Wang et al. [9] dataset from https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset: 456 CD-HIT-filtered sequences (30% identity cutoff; 152 positive R proteins, 304 negative) split into 364 training and 92 independent test samples. Run prPred-DRLF (Python 3.7 toolkit using fused BiLSTM 3605D + UniRep 1900D features with an LGBM classifier) and the prior prPred predictor on the independent test FASTA to get R/non-R predictions. Run HMMER profile alignment scoring on the positive training set only, because most negative training set sequences are annotated incompletely under HMMER. Compute precision, recall, accuracy, F1-score, and AUC for all three methods from TP/FP/TN/FN counts and ROC analysis, then compute the deltas of prPred-DRLF over prPred.

## Related tools

- **prPred-DRLF** (Primary predictor evaluated on the independent test set; encodes sequences as deep representation learning features (BiLSTM + UniRep fused) and classifies R vs non-R with LGBM) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred** (State-of-the-art baseline predictor run on the same independent test sequences to compute ACC/F1/AUC improvement deltas)
- **HMMER** (Profile alignment baseline scored on the positive training set only (negative training sequences are annotated incompletely under HMMER))
- **LightGBM (LGBM)** (Classifier producing the final R/non-R predictions evaluated on the independent test set)
- **Python 3.7** (Runtime environment for prPred-DRLF (requirements: joblib 1.0.1, tape_proteins 0.5, numpy 1.19.2, pandas 1.2.0, Bio 0.4.1, lightgbm 2.3.0)) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- prPred-DRLF deltas over prPred must match the reported improvements: ACC +2.20%, F1-score +4.29%, AUC +4.91%
- prPred-DRLF independent test metrics must reach ACC 0.956, F1-score 0.933, and AUC 0.997
- prPred-DRLF recall must exceed HMMER recall, consistent with reported overall improvement in accuracy, F1-score, AUC, and recall
- Test set must contain exactly 92 sequences (364 training + 92 test = 456 post-CD-HIT sequences; 152 positive, 304 negative)
- HMMER comparison restricted to the positive training set — verify no negative training sequences were fed to HMMER

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program, limiting computational efficiency
- Most negative training set sequences are annotated incompletely under HMMER, so the HMMER baseline cannot be evaluated on negatives

## Evidence

- [abstract] Independent test achieved accuracy 0.956, F1-score 0.933, and AUC 0.997: [evidence span withheld — anchor status: not_found]
- [methods] Dataset split into 364 training and 92 independent test samples: [evidence span withheld — anchor status: not_found]
