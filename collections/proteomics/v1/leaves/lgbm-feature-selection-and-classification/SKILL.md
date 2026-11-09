---
name: lgbm-feature-selection-and-classification
description: Use when you have a high-dimensional fused protein embedding (e.g., 5505D
  BiLSTM+UniRep vectors from FASTA sequences) and need feature selection plus a strong
  classifier for binary protein-family prediction with accuracy/F1/AUC evaluation.
when_to_use_negative:
- Input is already a compact, curated feature table — LGBM importance ranking over
  a 5505D fused embedding is unnecessary and may discard informative dimensions
- Sequences are highly redundant — apply CD-HIT with a 30% identity cutoff first or
  metrics will be inflated by near-duplicate sequences
- HMMER-style profile/domain annotation is the goal; note the article's caveat that
  HMMER evaluation was restricted to the positive training set because most negative-set
  sequences were incompletely annotated
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LightGBM (LGBM)
  role: Serves as both the feature-selection method (importance ranking of the 5505D
    fused vector) and the final classifier for R vs non-R protein prediction
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred-DRLF
  role: Open-source Python 3.7 toolkit that packages the embedding, LGBM feature selection,
    classification, and prediction steps; run via prPred-DRLF.py with -i FASTA and
    -o CSV output
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: eFeature
  role: Generates the BiLSTM, UniRep, and TAPE-BERT numerical embedding vectors that
    form the LGBM input features
- name: CD-HIT
  role: Pre-filters the sequence dataset with a 30% identity cutoff to remove redundancy
    before feature extraction and classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:9ea4b5e6e0b41bd095509ced35b8aea516504d34a69ebe6276af0871db4db19a
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lgbm-feature-selection-and-classification

## Summary

Applies a Light Gradient Boosting Machine (LGBM) both as a feature-importance ranker to extract an optimal subset from high-dimensional fused protein embeddings and as the final classifier for plant R protein (R vs non-R) prediction, as in prPred-DRLF.

## When to use

Use when you have a high-dimensional fused protein embedding (e.g., 5505D BiLSTM+UniRep vectors from FASTA sequences) and need feature selection plus a strong classifier for binary protein-family prediction with accuracy/F1/AUC evaluation.

## When NOT to use

- Input is already a compact, curated feature table — LGBM importance ranking over a 5505D fused embedding is unnecessary and may discard informative dimensions
- Sequences are highly redundant — apply CD-HIT with a 30% identity cutoff first or metrics will be inflated by near-duplicate sequences
- HMMER-style profile/domain annotation is the goal; note the article's caveat that HMMER evaluation was restricted to the positive training set because most negative-set sequences were incompletely annotated

## Inputs

- FASTA file of protein sequences (e.g., ./dataset/test_data.fasta) pre-filtered with CD-HIT at 30% identity
- Fused numerical feature vectors: BiLSTM 3605D + UniRep 1900D = 5505D (optionally TAPE-BERT 768D) generated via eFeature
- Binary labels for training samples (1 = R protein, 0 = non-R protein); dataset of 456 sequences (152 positive, 304 negative), 364 train / 92 independent test

## Outputs

- Ranked feature subset selected by LGBM feature importance from the 5505D fused vector
- predict_result.csv with R protein / non-R protein predictions for the independent test set
- Evaluation metrics: accuracy, precision, recall, F1-score, and AUC (reference: 0.956 / 0.967 / 0.905 / 0.933 / 0.997)

## How to apply

Encode protein sequences into numerical vectors with deep representation models (BiLSTM 3605D, UniRep 1900D, optionally TAPE-BERT 768D) via eFeature, and concatenate into a fused 5505D vector. Apply LGBM feature selection to rank features by importance and extract the optimal feature subset (comparable alternatives: RF and MRMD3.0), comparing selectors by fivefold cross-validation on the 364 training samples. Train the LGBM classifier on the selected features with labels 1 = R protein, 0 = non-R protein, then run the packaged script on the 92-sample independent test FASTA to produce a prediction CSV. Evaluate with precision, recall, accuracy, F1-score, and AUC; the reference result on the BiLSTM+UniRep fused features with LGBM is ACC 0.956, precision 0.967, recall 0.905, F1 0.933, AUC 0.997. Use the pinned environment (python 3.7, lightgbm-2.3.0, joblib 1.0.1, tape_proteins 0.5, numpy 1.19.2, pandas 1.2.0, Bio 0.4.1) since the README specifies exact versions.

## Related tools

- **LightGBM (LGBM)** (Serves as both the feature-selection method (importance ranking of the 5505D fused vector) and the final classifier for R vs non-R protein prediction) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred-DRLF** (Open-source Python 3.7 toolkit that packages the embedding, LGBM feature selection, classification, and prediction steps; run via prPred-DRLF.py with -i FASTA and -o CSV output) — https://github.com/Wangys-prog/prPred-DRLF
- **eFeature** (Generates the BiLSTM, UniRep, and TAPE-BERT numerical embedding vectors that form the LGBM input features)
- **CD-HIT** (Pre-filters the sequence dataset with a 30% identity cutoff to remove redundancy before feature extraction and classification)

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Independent test metrics match the reported values: accuracy 0.956, precision 0.967, recall 0.905, F1-score 0.933, AUC 0.997 within a small tolerance
- Fused BiLSTM+UniRep features outperform individual BiLSTM (3605D), UniRep (1900D), and TAPE-BERT (768D) features in fivefold cross-validation with the LGBM classifier
- Output CSV (predict_result.csv) contains binary predictions (R protein / non-R protein) for all 92 independent test sequences
- Training set contains exactly 364 samples and test set 92 samples from the 456-sequence CD-HIT-filtered dataset (152 positive, 304 negative)
- LGBM feature selection yields a feature subset that beats alternatives (RF, MRMD3.0) by cross-validation

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program, limiting computational efficiency
- GPU acceleration is optional but recommended (README: 'If your computer has GPU, it will be faster'); environment is pinned to Python 3.7 with lightgbm-2.3.0 and specific torch versions
- HMMER baseline evaluation could only be applied to the positive training set because most negative-set sequences were incompletely annotated, limiting baseline comparability

## Evidence

- [abstract] Independent test performance of the LGBM model: [evidence span withheld — anchor status: not_found]
