---
name: binary-classifier-benchmarking
description: Use when two or more binary classifier variants (e.g., a stacked ensemble
  vs. its base learners) must be compared on the same test set, especially under class
  imbalance where AUPRC and recall at probability thresholds matter more than accuracy.
when_to_use_negative:
- Input is already a model comparison table or published metrics and the goal is to
  interpret, rather than re-run, the benchmark
- The task is multi-class classification rather than the binary positive/negative
  phosphosite prediction per residue model (ST, Y)
- No stratified train/validation/test split exists and cannot be constructed, since
  base-learner and ensemble metrics would not be comparable on the same held-out test
  set
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CatBoost
  role: Gradient-boosting tree base classifier trained both inside the stacking ensemble
    and independently for the benchmark; one variant uses auto_class_weights='balanced',
    the other default equal weights
  repo: https://github.com/eporetsky/PhosBoost
- name: scikit-learn
  role: Provides the stacking classifier, logistic regression metaclassifier, 5-fold
    cross-validated prediction approach, and train_test_split (shuffle=True, stratify=y)
    for the stratified splits
- name: PhosBoost
  role: Stacking classifier being benchmarked against its independently trained CatBoost
    base classifiers; the compared model
  repo: https://github.com/eporetsky/PhosBoost
- name: BayesianOptimization
  role: Bayesian hyperparameter tuning per classifier variant (100 iterations, optimizing
    F1) so all compared models receive equally tuned hyperparameters before benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:b748756ce4354e25e9c50d07fff8ee2573bec0a44e661963e2d112d1af4e9e28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - BayesianOptimization
  - CatBoost
  - PhosBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# binary-classifier-benchmarking

## Summary

Benchmark binary classifiers on identical data splits by independently training each model variant outside the ensemble and comparing test-set metrics (AUPRC, AUROC, recall, F1). Used here to test whether the PhosBoost stacking classifier outperforms its own CatBoost base classifiers.

## When to use

Use when two or more binary classifier variants (e.g., a stacked ensemble vs. its base learners) must be compared on the same test set, especially under class imbalance where AUPRC and recall at probability thresholds matter more than accuracy.

## When NOT to use

- Input is already a model comparison table or published metrics and the goal is to interpret, rather than re-run, the benchmark
- The task is multi-class classification rather than the binary positive/negative phosphosite prediction per residue model (ST, Y)
- No stratified train/validation/test split exists and cannot be constructed, since base-learner and ensemble metrics would not be comparable on the same held-out test set

## Inputs

- Labeled phosphosite embedding vectors (ProtT5-XL-U50 residue vectors plus protein-wise average vectors for S/T/Y residues) with the y label column
- Stratified 60%-20%-20% train/validation/test split files for the ST and Y residue models (train_test_split with shuffle=True, stratify=y)
- Tuned hyperparameter files per variant from data/params/ (n_estimators, depth, learning_rate for balanced and equal weight classifiers)

## Outputs

- Trained independent CatBoost classifier joblib/pkl models (e.g., Arabidopsi.qPTMplants.ST.joblib.pkl) for balanced and equal weight variants
- Stacked-vs-base comparison table (phosboost_model_comparison.csv) with AUPRC, AUROC, recall at probability thresholds, and F1 per classifier per residue model
- Trained PhosBoost stacking classifier models (ST and Y) as joblib.pkl files

## How to apply

Start from a stratified 60%-20%-20% train/validation/test split and tune each model variant separately before comparison: BayesianOptimization over 100 iterations optimizing F1, with CatBoost hyperparameters sampled over n_estimators 50-2000, depth 2-10, and learning_rate 0.05-0.5. Train the base classifiers independently (one with auto_class_weights='balanced' for label imbalance, one with default equal weights) outside the stacking classifier, then train the stacking classifier (logistic regression metaclassifier fit on 5-fold cross-validated predicted probabilities of the two CatBoost base classifiers). Evaluate all variants on the same held-out test set — here the Arabidopsis qPTMplants ST and Y ProtT5 embedding test sets — computing AUPRC, AUROC, recall at probability thresholds, and F1. Judge the comparison by whether the stacked model's AUPRC exceeds both base classifiers (PhosBoost .54 vs equal-weight .53 vs balanced .48) and whether recall is higher at all probability thresholds, as compiled into a stacked-vs-base table (phosboost_model_comparison.csv).

## Related tools

- **CatBoost** (Gradient-boosting tree base classifier trained both inside the stacking ensemble and independently for the benchmark; one variant uses auto_class_weights='balanced', the other default equal weights) — https://github.com/eporetsky/PhosBoost
- **scikit-learn** (Provides the stacking classifier, logistic regression metaclassifier, 5-fold cross-validated prediction approach, and train_test_split (shuffle=True, stratify=y) for the stratified splits)
- **PhosBoost** (Stacking classifier being benchmarked against its independently trained CatBoost base classifiers; the compared model) — https://github.com/eporetsky/PhosBoost
- **BayesianOptimization** (Bayesian hyperparameter tuning per classifier variant (100 iterations, optimizing F1) so all compared models receive equally tuned hyperparameters before benchmarking)

## Examples

```
sbatch tune.sh name ST 100 balanced
```

## Evaluation signals

- On the A. thaliana qPTMplants test set, expected ordering for S/T AUPRC is PhosBoost (.54) > equal-weight CatBoost (.53) > balanced-weight CatBoost (.48)
- All three S/T classifiers should land at a similar AUROC of approximately .88, so AUROC alone should not drive the comparison
- PhosBoost should show higher recall than the independent base classifiers at every probability threshold (headline recall .78 for S/T and .6 for Y)
- Hyperparameter tuning logs in log/<batch_id> must exist separately for each variant (ST/Y x balanced/equal) with 100 BayesianOptimization iterations optimizing F1
- The comparison CSV (phosboost_model_comparison.csv) should contain paired rows for stacked and independent models for both ST and Y residue models

## Limitations

- PhosBoost's higher recall comes with a precision-recall tradeoff: an increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- The mechanism behind the stacking benefit is a hypothesis, not demonstrated: the authors only hypothesize that the balanced-weight base classifier helps address class imbalance
- Label uncertainty (many phosphosites remain undetected by high-throughput phosphoproteomics) limits the reliability of negative labels and therefore of all benchmark metrics

## Evidence

- [results] The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency: "The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency"
- [methods] Stratified 60-20-20 split: "randomly split into stratified training, validation, and testing sets using a 60%–20%–20%, split, respectively"
- [readme] Bayesian hyperparameter tuning over 100 iterations optimizing F1: "the Bayesian hyperparameter tuning ran over 100 iterations, optimizing for the F1 score at each iteration."
- [results] Similar AUROC of .88 across all three classifiers: "all three classifiers had a similar area under the receiver operating characteristic (AUROC) score of .88"
