---
name: protein-sequence-prediction-benchmarking
description: Use when a new sequence-classification predictor must be compared to
  prior tools (e.g., prPred, HMMER) on the same labeled FASTA test set to quantify
  ACC/F1/AUC/recall improvements.
when_to_use_negative:
- The negative training set sequences are annotated incompletely under HMMER — HMMER
  evaluation must then be restricted to the positive training set only, not the full
  negative set
- The goal is training a new model or selecting feature subsets (RF, LGBM, MRMD3.0)
  rather than comparing an already-trained predictor against baselines
- The input is raw amino-acid sequences needing feature embedding (BiLSTM/UniRep/TAPE-BERT)
  rather than a head-to-head predictor comparison
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0084
tools:
- name: prPred-DRLF
  role: New predictor under evaluation; run on the independent test FASTA to produce
    R/non-R predictions
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred
  role: State-of-the-art baseline predictor run on the same independent test sequences
    to compute performance deltas
- name: HMMER
  role: Profile-alignment baseline; applied only to the positive training set because
    most negative training set sequences are annotated incompletely
- name: Python 3.7
  role: Environment for running prPred-DRLF and computing metrics via Python scripting
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:53ab872ac5f72ad21948fca26b0ae7109081a2c16e38d7d1efa41eac0d2e4cd5
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# protein-sequence-prediction-benchmarking

## Summary

Benchmarks a protein-sequence predictor against published baselines on a shared independent test set using TP/FP/TN/FN-derived precision, recall, ACC, F1-score, and ROC-AUC, plus explicit performance deltas over the prior predictor (prPred).

## When to use

Use when a new sequence-classification predictor must be compared to prior tools (e.g., prPred, HMMER) on the same labeled FASTA test set to quantify ACC/F1/AUC/recall improvements.

## When NOT to use

- The negative training set sequences are annotated incompletely under HMMER — HMMER evaluation must then be restricted to the positive training set only, not the full negative set
- The goal is training a new model or selecting feature subsets (RF, LGBM, MRMD3.0) rather than comparing an already-trained predictor against baselines
- The input is raw amino-acid sequences needing feature embedding (BiLSTM/UniRep/TAPE-BERT) rather than a head-to-head predictor comparison

## Inputs

- Independent test dataset in FASTA format (92 plant protein sequences) from the Wang et al. [9] dataset
- Positive training set in FASTA format (152 R protein sequences) for HMMER profile scoring
- Pre-trained prPred-DRLF model script (prPred-DRLF.py) and the baseline prPred predictor
- Binary R/non-R labels (positive/negative) for computing the confusion matrix

## Outputs

- Confusion matrix counts (TP/FP/TN/FN) per method
- Table of precision, recall, accuracy (ACC), F1-score, and AUC for prPred-DRLF, prPred, and HMMER
- Performance deltas of prPred-DRLF over prPred (ACC +2.20%, F1-score +4.29%, AUC +4.91%)
- ROC curve / AUC comparison plot data

## How to apply

Load the independent test set (92 sequences) and the positive training set (152 R proteins) from the Wang et al. [9] plant R protein dataset hosted at https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset. Run the new predictor (prPred-DRLF, a Python 3.7 toolkit) and the baseline prPred on the same independent test FASTA sequences to obtain R/non-R protein predictions. For HMMER, restrict profile alignment scoring to the positive training set only, because most negative training set sequences are annotated incompletely under HMMER. Compute precision, recall, accuracy, F1-score, and AUC for all methods using standard TP/FP/TN/FN definitions and ROC analysis, then report the prPred-DRLF deltas over prPred (ACC +2.20%, F1-score +4.29%, AUC +4.91%) and the recall comparison versus HMMER.

## Related tools

- **prPred-DRLF** (New predictor under evaluation; run on the independent test FASTA to produce R/non-R predictions) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred** (State-of-the-art baseline predictor run on the same independent test sequences to compute performance deltas)
- **HMMER** (Profile-alignment baseline; applied only to the positive training set because most negative training set sequences are annotated incompletely)
- **Python 3.7** (Environment for running prPred-DRLF and computing metrics via Python scripting) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Reproduced deltas over prPred must match the reported improvements: ACC +2.20%, F1-score +4.29%, AUC +4.91%
- prPred-DRLF absolute independent-test values should reach ACC 0.956, F1-score 0.933, and AUC 0.997
- prPred-DRLF recall must exceed HMMER recall, consistent with the reported overall improvement in accuracy, F1-score, AUC, and recall
- Test-set size must be exactly 92 sequences (456 total after CD-HIT 30% identity filtering: 152 positive, 304 negative, 364 training / 92 test)
- Predictions must be binary R/non-R labels saved as CSV when running the prPred-DRLF.py script

## Limitations

- HMMER evaluation is restricted to the positive training set only, so the HMMER comparison is not on the full labeled test set
- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program, limiting computational efficiency
- The web server homepage is shown (Figure 6) but no access URL is given in the article text (README cites http://lab.malab.cn/soft/prPred-DRLF)

## Evidence

- [abstract] The model was evaluated using an independent test achieving an accuracy of 0.956, F1-scoreof0.933,andareaunderthereceiveroperatingcharacteristic(ROC)curve(AUC)of 0.997.: [evidence span withheld — anchor status: not_found]
- [readme] usage: Script for predicting plant R protein using deep representation learning features [-h] [-i I] [-o O] ... input sequences in Fasta format ... path to saved CSV file: [evidence span withheld — anchor status: not_found]
