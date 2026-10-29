---
name: class-imbalanced-protein-dataset-evaluation
description: Use when benchmarking protein sequence classifiers on a dataset with
  unequal positive/negative counts (e.g., 1:2 ratio like 152 R proteins vs 304 non-R),
  where accuracy alone can mask weak minority-class recall and F1/AUC must be reported
  alongside it.
when_to_use_negative:
- When the goal is generating embeddings or predictions from already-trained models
  rather than benchmarking classifier/feature-selection combinations (use the prPred-DRLF
  prediction script directly instead)
- When the dataset is balanced and large enough that accuracy alone is a sufficient
  comparison metric, making the full precision/recall/F1/AUC grid unnecessary
- When negative-class sequences are incompletely annotated such that only positive-set
  evaluation is meaningful (as the article found for HMMER comparison)
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: prPred-DRLF
  role: Provides the 456-sequence plant R protein dataset, BiLSTM+UniRep 5505D feature
    vectors, and the LGBM-based prediction/evaluation pipeline whose fivefold cross-validation
    performance is benchmarked
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Reference best classifier and feature-selection method, achieving the top
    cross-validated ACC 0.970 and F1 0.953 in the 5-fold CV benchmark
- name: MRMD3.0
  role: One of three feature-selection (feature importance ranking) methods compared
    in the cross-validation ablation
  repo: http://lab.malab.cn/soft/MRMD3.0/index.html
- name: Python 3.7
  role: Runtime environment for the open-source prPred-DRLF toolkit (see README conda/pip
    setup with requirements.txt)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:f32f13bb9b93ac514e8c97b480bea4eefaa519adfd81f771dbfe7f0e3c887de6
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
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# class-imbalanced-protein-dataset-evaluation

## Summary

Evaluates protein classification models on a mildly class-imbalanced sequence dataset (152 positive R proteins vs 304 negatives) using fivefold cross-validation with precision, recall, accuracy (ACC), F1-score, and ROC-AUC rather than accuracy alone, so minority-class performance is captured.

## When to use

Use when benchmarking protein sequence classifiers on a dataset with unequal positive/negative counts (e.g., 1:2 ratio like 152 R proteins vs 304 non-R), where accuracy alone can mask weak minority-class recall and F1/AUC must be reported alongside it.

## When NOT to use

- When the goal is generating embeddings or predictions from already-trained models rather than benchmarking classifier/feature-selection combinations (use the prPred-DRLF prediction script directly instead)
- When the dataset is balanced and large enough that accuracy alone is a sufficient comparison metric, making the full precision/recall/F1/AUC grid unnecessary
- When negative-class sequences are incompletely annotated such that only positive-set evaluation is meaningful (as the article found for HMMER comparison)

## Inputs

- Labeled protein sequence dataset in FASTA format with class labels (here: 456 sequences, 152 positive R proteins / 304 negative, split 364 train / 92 independent test)
- Numerical feature matrix per sequence (fused BiLSTM 3605D + UniRep 1900D = 5505D from the prPred-DRLF GitHub dataset)
- List of candidate feature-selection methods (RF, LGBM, MRMD3.0) and classifiers (SVM, LGBM, RF, ET)

## Outputs

- Cross-validation metrics table (e.g., cv_benchmark_results.csv) with precision, recall, ACC, F1-score, and AUC for every feature-selection × classifier combination
- ROC curves (true positive rate vs false positive rate) and AUC values per model
- Identified best configuration (LGBM feature selection + LGBM classifier) with reference metrics ACC 0.970 / F1 0.953 / AUC 0.988
- Independent-test evaluation metrics (expected ≈ ACC 0.956, F1 0.933, AUC 0.997)

## How to apply

Take the CD-HIT-filtered (30% identity cutoff) dataset of 456 sequences — 152 positive, 304 negative — split 364 training / 92 independent test, and run fivefold cross-validation on the training set to examine each model's effectiveness. Embed sequences first (prPred-DRLF's BiLSTM 3605D + UniRep 1900D fused 5505D vectors), then for every feature-selection (RF, LGBM, MRMD3.0) × classifier (SVM, LGBM, RF, ET) combination compute precision, recall, accuracy (ACC), F1-score, and AUC from the ROC curve (true positive rate vs false positive rate). Because positives are outnumbered ~2:1, treat recall and F1 as the discriminating metrics; the reference best configuration (LGBM feature selection + LGBM classifier) reached cross-validated ACC 0.970, F1 0.953, AUC 0.988, and independent-test ACC 0.956, F1 0.933, AUC 0.997. Save the full cross-validation metrics table (e.g., cv_benchmark_results.csv) so all combinations remain comparable.

## Related tools

- **prPred-DRLF** (Provides the 456-sequence plant R protein dataset, BiLSTM+UniRep 5505D feature vectors, and the LGBM-based prediction/evaluation pipeline whose fivefold cross-validation performance is benchmarked) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Reference best classifier and feature-selection method, achieving the top cross-validated ACC 0.970 and F1 0.953 in the 5-fold CV benchmark)
- **MRMD3.0** (One of three feature-selection (feature importance ranking) methods compared in the cross-validation ablation) — http://lab.malab.cn/soft/MRMD3.0/index.html
- **Python 3.7** (Runtime environment for the open-source prPred-DRLF toolkit (see README conda/pip setup with requirements.txt))

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Reproduced best configuration should match reported cross-validated LGBM + LGBM performance: ACC 0.970, F1 0.953, AUC 0.988
- Independent test of the best model should reproduce ≈ ACC 0.956, F1 0.933, AUC 0.997
- Training/test split must preserve 364 training and 92 independent test samples from the 456-sequence CD-HIT-filtered dataset (152 positive, 304 negative)
- Metrics table must contain precision, recall, accuracy (ACC), F1-score, and AUC for all 3 feature-selection × 4 classifier = 12 combinations
- Recall on the minority (positive R protein) class should be reported and consistent with the high F1/AUC, not masked by accuracy

## Limitations

- Class imbalance (~1:2 positive:negative) means accuracy can be inflated; recall and F1 are the more informative metrics for the minority class
- The dataset is small (456 sequences), so cross-validation variance may be non-trivial
- Deep representation learning features require an elevated level of computing resources; parallel computing is not incorporated into the program
- For HMMER-style comparison, only positive training sequences could be evaluated because most negative training set sequences were annotated incompletely

## Evidence

- [methods] Fivefold cross-validation on the dataset is used to examine the effectiveness of each model: "Fivefold cross-validation on the dataset is used to examine the effectiveness of each model"
- [methods] Dataset composition: [evidence span withheld — anchor status: not_found]
- [methods] Evaluation measures: "four evaluation measures are used, including precision, recall, accuracy (ACC), and F1-score"
- [abstract] Independent test performance reference: [evidence span withheld — anchor status: not_found]
- [readme] Runtime environment for the evaluation pipeline: "prPred-DRLF is an open-source Python-based toolkit, which operates depending on the Python environment (##Python Version 3.7)."
