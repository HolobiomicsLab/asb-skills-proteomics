---
name: gradient-boosting-feature-ranking
description: Use when a high-dimensional fused protein-embedding feature vector (e.g.,
  5505D BiLSTM+UniRep) must be ranked and reduced by feature importance before training
  classifiers such as SVM, LGBM, RF, or ET.
when_to_use_negative:
- Input is raw FASTA sequences that have not yet been embedded — run BiLSTM/UniRep
  embedding first; this skill operates on numerical feature vectors
- The pipeline's goal is homology-based annotation (HMMER profiling) rather than machine-learning
  feature reduction
- The feature table is already low-dimensional, so importance-based subsetting offers
  no reduction benefit
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LightGBM (LGBM)
  role: Gradient boosting machine whose feature importance values rank the 5505D fused
    BiLSTM+UniRep vector; also serves as the classifier used to validate the LGBM-selected
    subset (5-fold CV ACC 0.970)
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: MRMD3.0
  role: Max-relevance-max-distance feature selection baseline compared against the
    LGBM ranking
  repo: http://lab.malab.cn/soft/MRMD3.0/index.html
- name: prPred-DRLF
  role: Open-source Python 3.7 toolkit providing the dataset, embeddings, and prediction
    script that surround the feature-ranking step
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:596449bc7df29778ebfc87c50575d22e892cf3d4c64a924e291cd95fe15b60d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - LightGBM (LGBM)
  - MRMD3.0
  - Python
  - prPred-DRLF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gradient-boosting-feature-ranking

## Summary

Ranks features of a high-dimensional protein-embedding matrix by importance values from a LightGBM (LGBM) model, comparing the LGBM ranking against RF and MRMD3.0 rankings to select an optimal feature subset for plant R protein classification.

## When to use

Use when a high-dimensional fused protein-embedding feature vector (e.g., 5505D BiLSTM+UniRep) must be ranked and reduced by feature importance before training classifiers such as SVM, LGBM, RF, or ET.

## When NOT to use

- Input is raw FASTA sequences that have not yet been embedded — run BiLSTM/UniRep embedding first; this skill operates on numerical feature vectors
- The pipeline's goal is homology-based annotation (HMMER profiling) rather than machine-learning feature reduction
- The feature table is already low-dimensional, so importance-based subsetting offers no reduction benefit

## Inputs

- Fused BiLSTM + UniRep numerical feature matrix (5505D: 3605D + 1900D) for 364 training sequences with 1/0 R-protein labels
- Raw protein sequences in FASTA format (e.g., ./dataset/test_data.fasta) to be embedded upstream of feature ranking
- Trained LGBM (lightgbm 2.3.0, Python 3.7) model object from which feature importances are extracted

## Outputs

- Feature-importance ranking per method (LGBM, RF, MRMD3.0) over the 5505D feature vector
- Selected optimal feature subset(s) used as classifier input
- Cross-validation metrics table (precision, recall, ACC, F1, AUC per feature-selection × classifier combination, e.g., cv_benchmark_results.csv)

## How to apply

Start from the fused BiLSTM (3605D) + UniRep (1900D) feature matrix built from the 456-sequence plant R protein dataset (152 positive, 304 negative after CD-HIT 30% identity filtering; 364 training samples). Rank features by importance values generated with three methods — RF, LGBM, and MRMD3.0 (http://lab.malab.cn/soft/MRMD3.0/index.html) — and select the optimal subset from each ranking. Feed each selected subset into four classifiers (SVM, LGBM, RF, extra tree) under 5-fold cross-validation, computing precision, recall, ACC, F1-score, and AUC. The decision point is choosing the feature-selection × classifier combination with the best cross-validated metrics; in this paper LGBM feature selection paired with the LGBM classifier won (ACC 0.970, F1 0.953, AUC 0.988), so the LGBM importance ranking is the preferred default. Persist all combination metrics (e.g., to cv_benchmark_results.csv) to justify the ranking choice.

## Related tools

- **LightGBM (LGBM)** (Gradient boosting machine whose feature importance values rank the 5505D fused BiLSTM+UniRep vector; also serves as the classifier used to validate the LGBM-selected subset (5-fold CV ACC 0.970)) — https://github.com/Wangys-prog/prPred-DRLF
- **MRMD3.0** (Max-relevance-max-distance feature selection baseline compared against the LGBM ranking) — http://lab.malab.cn/soft/MRMD3.0/index.html
- **prPred-DRLF** (Open-source Python 3.7 toolkit providing the dataset, embeddings, and prediction script that surround the feature-ranking step) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- LGBM-selected features + LGBM classifier should reach 5-fold CV accuracy 0.970, F1-score 0.953, and AUC 0.988 on the 364-sequence training set
- Independent test of the final model should reproduce ACC 0.956, F1 0.933, AUC 0.997
- Ranked matrix dimension must derive from the fused 5505D vector (3605D BiLSTM + 1900D UniRep), not the 768D TAPE-BERT features
- The LGBM ranking should outperform or match the RF and MRMD3.0 rankings when benchmarked across the same four classifiers (SVM, LGBM, RF, ET) with identical CV folds
- Metrics table must contain all five measures (precision, recall, ACC, F1, AUC) for every feature-selection × classifier combination

## Limitations

- Deep representation learning features used upstream require an elevated level of computing resources (stated limitation)
- No parallel computing is incorporated; the authors note they plan to add it for high computational efficiency
- Environment is version-pinned (Python 3.7, lightgbm 2.3.0, numpy 1.19.2, pandas 1.2.0); newer LGBM versions may not reproduce the ranking
- Results depend on the Wang et al. [9] 456-sequence plant R protein dataset (152 positive, 304 negative); applicability to other labels/sizes is unverified

## Evidence

- [intro] Three feature selection techniques, including random forest (RF), LGBM, and max-relevance-max-distance (MRMD), were applied to select an optimal feature subset: [evidence span withheld — anchor status: not_found]
