---
name: hmm-profile-alignment-scoring
description: Use when benchmarking a plant R protein predictor against the HMMER profile
  alignment baseline, especially when negative training set sequences are annotated
  incompletely under HMMER so scoring must be restricted to positive sequences.
when_to_use_negative:
- When you need negative-class metrics (TN, specificity, full ACC/F1/AUC) from HMMER
  itself — most negative training set sequences are annotated incompletely under HMMER,
  so those numbers would be unreliable
- When the goal is R protein prediction rather than baseline comparison — use the
  prPred-DRLF predictor (LGBM on fused BiLSTM + UniRep features) instead
- When input data is not protein amino acid sequences (e.g., nucleotide or feature
  tables) — HMMER profile alignment requires protein FASTA input
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: HMMER
  role: Performs the profile hidden Markov model alignment scoring used as the traditional-method
    baseline for R protein detection
- name: prPred-DRLF
  role: The deep representation learning predictor (LGBM on fused BiLSTM + UniRep
    features) whose recall is compared against the HMMER baseline
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred
  role: The prior k-spaced amino acid (group) pairs predictor evaluated alongside
    HMMER on the same independent test dataset (92 sequences)
- name: Python 3.7
  role: Environment for running prPred-DRLF and computing the evaluation metrics used
    in the comparison against HMMER
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:a980349af0c84c360cd36a979cadb70beda1aa1f88f2a042996e484c26baf83b
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# HMM profile alignment scoring

## Summary

Benchmark baseline for plant R protein identification in which HMMER profile hidden Markov models align and score sequences to detect R proteins, serving as the traditional-method comparator against prPred-DRLF and prPred. In the prPred-DRLF evaluation, HMMER scoring is applied only to the positive training set because negative training sequences are incompletely annotated.

## When to use

Use when benchmarking a plant R protein predictor against the state-of-the-art HMMER profile alignment baseline, especially when negative training set sequences are annotated incompletely under HMMER so scoring must be restricted to positive sequences.

## When NOT to use

- When you need negative-class metrics (TN, specificity, full ACC/F1/AUC) from HMMER itself — most negative training set sequences are annotated incompletely under HMMER, so those numbers would be unreliable
- When the goal is R protein prediction rather than baseline comparison — use the prPred-DRLF predictor (LGBM on fused BiLSTM + UniRep features) instead
- When input data is not protein amino acid sequences (e.g., nucleotide or feature tables) — HMMER profile alignment requires protein FASTA input

## Inputs

- Positive training set of 152 plant R protein sequences in FASTA format (from the Wang et al. [9] dataset, https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset)
- HMMER profile HMM library for R protein / domain profiles

## Outputs

- HMMER alignment scores / hit calls per input sequence (R protein vs. no hit)
- Precision and recall for HMMER computed from TP/FN over the 152 positive training sequences
- Comparison table of prPred-DRLF vs. prPred vs. HMMER metrics (precision, recall, accuracy, F1-score, AUC)

## How to apply

Run HMMER profile alignment scoring on the positive training set (152 R proteins from the Wang et al. [9] dataset hosted at https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset) rather than on the full 456-sequence dataset (152 positive, 304 negative), because most protein sequences from the negative training set are annotated incompletely using the HMMER method. Restricting to positives means recall for HMMER is computed from true positives and false negatives among the 152 positive sequences; metrics requiring true negatives (accuracy, specificity) are not comparable to full-dataset metrics from prPred-DRLF or prPred. Evaluate alongside prPred-DRLF and prPred using the standard TP/FP/TN/FN definitions and ROC analysis, computing precision, recall, accuracy, F1-score, and AUC where defined. Use the resulting recall as the primary comparison point: the validated outcome is that prPred-DRLF recall exceeds HMMER recall, contributing to the reported overall improvement in accuracy, F1-score, AUC, and recall.

## Related tools

- **HMMER** (Performs the profile hidden Markov model alignment scoring used as the traditional-method baseline for R protein detection)
- **prPred-DRLF** (The deep representation learning predictor (LGBM on fused BiLSTM + UniRep features) whose recall is compared against the HMMER baseline) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred** (The prior k-spaced amino acid (group) pairs predictor evaluated alongside HMMER on the same independent test dataset (92 sequences))
- **Python 3.7** (Environment for running prPred-DRLF and computing the evaluation metrics used in the comparison against HMMER) — https://github.com/Wangys-prog/prPred-DRLF

## Evaluation signals

- HMMER scoring must be restricted to the positive training set (152 sequences); any TN-dependent HMMER metrics derived from negative training sequences are invalid because negatives are incompletely annotated
- Recall for HMMER must be computed from TP/FN over the 152 positive training sequences using the standard TP/FP/TN/FN definitions
- prPred-DRLF recall must exceed HMMER recall, consistent with the reported overall improvement in accuracy, F1-score, AUC, and recall
- The comparison must report prPred-DRLF deltas over prPred of ACC +2.20%, F1-score +4.29%, and AUC +4.91% on the 92-sequence independent test set
- prPred-DRLF independent test values should match accuracy 0.956, F1-score 0.933, and AUC 0.997

## Limitations

- Most protein sequences from the negative training set are annotated incompletely using the HMMER method, so HMMER cannot be evaluated on the full training set and negative-class metrics are unavailable for this baseline
- Because only positives are scored, HMMER recall is not directly comparable to full-dataset ACC/F1/AUC values reported for prPred-DRLF and prPred
- HMMER is a traditional feature/profile method; prPred-DRLF was proposed precisely because existing predictors rely on traditional feature extraction methods, which the paper shows are outperformed

## Evidence
