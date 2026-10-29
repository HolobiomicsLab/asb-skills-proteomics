---
name: stacking-ensemble-model-training
description: Use when a labeled, class-imbalanced biosequence dataset has per-residue
  feature embeddings (e.g., ProtT5 residue plus protein-average vectors) and a stacked
  binary classifier must be built and compared against its base classifiers.
when_to_use_negative:
- The input is already a feature table from a trained model or the goal is only to
  score new sequences with an existing trained .pkl model — use the prediction skill
  instead of retraining
- The dataset lacks a y label column or has not yet been split into train/validation/test
  sets — complete data labeling and splitting first
- The problem is multi-class or requires per-residue sequence labeling beyond binary
  phosphosite/non-phosphosite classification
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3510
tools:
- name: CatBoost
  role: Gradient-boosting tree base classifiers, one trained with auto_class_weights='balanced'
    and one with equal class weights, whose predicted probabilities feed the metaclassifier
  repo: https://github.com/eporetsky/PhosBoost
- name: scikit-learn
  role: Stacking framework providing the 5-fold cross-validated logistic regression
    metaclassifier and the train_test_split (shuffle=True, stratify=y) utility
  repo: https://github.com/eporetsky/PhosBoost
- name: BayesianOptimization
  role: Bayesian hyperparameter tuning (100 iterations, F1 objective) of n_estimators,
    depth, and learning_rate for each CatBoost base classifier
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosBoost
  role: Pipeline wrapper that assembles and trains the ST and Y stacking classifiers
    and saves them as joblib .pkl model files
  repo: https://github.com/eporetsky/PhosBoost
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:8789441aae7852f845d9884c5c106149c9a04ef64b25c47edd19e1d7149b409a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - BayesianOptimization
  - CatBoost
  - PhosBoost
  - ProtT5-XL-U50
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# stacking-ensemble-model-training

## Summary

Trains a stacking ensemble classifier in which two gradient-boosting tree base classifiers (CatBoost with balanced vs. equal class weights) supply cross-validated predicted probabilities to a scikit-learn logistic regression metaclassifier, as implemented in PhosBoost for phosphorylation prediction from ProtT5 embeddings.

## When to use

Use when a labeled, class-imbalanced biosequence dataset has per-residue feature embeddings (e.g., ProtT5 residue plus protein-average vectors) and a stacked binary classifier must be built and compared against its base classifiers.

## When NOT to use

- The input is already a feature table from a trained model or the goal is only to score new sequences with an existing trained .pkl model — use the prediction skill instead of retraining
- The dataset lacks a y label column or has not yet been split into train/validation/test sets — complete data labeling and splitting first
- The problem is multi-class or requires per-residue sequence labeling beyond binary phosphosite/non-phosphosite classification

## Inputs

- Labeled ProtT5-XL-U50 embedding pickle file (S/T/Y residue vectors plus protein-wise average vectors) with a y column (1=positive phosphosite, 0=negative)
- Stratified 60%-20%-20% train/validation/test split files for the ST and Y residue datasets
- Tuned CatBoost hyperparameter files (n_estimators, depth, learning_rate) from the log/ output of BayesianOptimization

## Outputs

- Trained PhosBoost stacking classifier .pkl model files saved under data/models/ (one per residue model, ST and Y)
- Independently trained balanced-weight and equal-weight CatBoost base classifiers for ablation comparison
- Stacked-vs-base performance comparison table (phosboost_model_comparison.csv) with metrics such as AUPRC, AUROC, recall, and F1 on the test set

## How to apply

First tune each base classifier separately with BayesianOptimization over 100 iterations optimizing F1, searching n_estimators (50–2000), depth (2–10), and learning_rate (0.05–0.5), once with auto_class_weights='balanced' and once with default equal weights, per residue model (ST and Y). Train two CatBoost (v1.1.1) base classifiers on the training split of the labeled ProtT5 embedding pickle. Assemble a scikit-learn StackingClassifier whose logistic regression metaclassifier (balanced class weights) is fit on 5-fold cross-validated predicted probabilities from the two base classifiers. Separately train the same two CatBoost classifiers outside the ensemble to quantify the stacking benefit, then evaluate all three on the held-out test set and compile a stacked-vs-base comparison table (e.g., phosboost_model_comparison.csv). Save the fitted ST and Y stacking models as .pkl files under data/models/.

## Related tools

- **CatBoost** (Gradient-boosting tree base classifiers, one trained with auto_class_weights='balanced' and one with equal class weights, whose predicted probabilities feed the metaclassifier) — https://github.com/eporetsky/PhosBoost
- **scikit-learn** (Stacking framework providing the 5-fold cross-validated logistic regression metaclassifier and the train_test_split (shuffle=True, stratify=y) utility) — https://github.com/eporetsky/PhosBoost
- **BayesianOptimization** (Bayesian hyperparameter tuning (100 iterations, F1 objective) of n_estimators, depth, and learning_rate for each CatBoost base classifier) — https://github.com/eporetsky/PhosBoost
- **PhosBoost** (Pipeline wrapper that assembles and trains the ST and Y stacking classifiers and saves them as joblib .pkl model files) — https://github.com/eporetsky/PhosBoost

## Examples

```
sbatch train.sh name ST name name  # trains the ST PhosBoost stacking model and saves the output .pkl to data/models/
```

## Evaluation signals

- Bayesian tuning log/ files show converged F1-score after 100 iterations for both balanced and equal weight variants of each residue model
- Stacked AUPRC exceeds both independent base classifiers (expected pattern: balanced-weight CatBoost ~0.48 < equal-weight CatBoost ~0.53 < PhosBoost stacking ~0.54 on the A. thaliana qPTMplants ST test set)
- PhosBoost achieves higher recall than the independent base classifiers at all probability thresholds
- ST test-set performance reaches the reported benchmark (recall ~0.78, AUPRC ~0.54) and Y reaches recall ~0.6
- Fitted model files are saved as .pkl (joblib) under data/models/ for both ST and Y residue models

## Limitations

- The mechanism of the stacking benefit is a stated hypothesis: the paper notes 'We hypothesize that the use of balanced class weights to train one base classifier helps address the class imbalance' — the ablation shows improved recall/AUPRC but does not demonstrate the underlying cause
- PhosBoost shows a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- Label uncertainty from undetected phosphosites in high-throughput phosphoproteomics limits negative-label reliability and constrains achievable performance

## Evidence

- [results] The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency: "The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency"
- [results] the logistic regression metaclassifier that was trained using a 5-fold cross-validated prediction approach: "the logistic regression metaclassifier that was trained using a 5‐fold cross‐validated prediction approach"
- [results] Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split: "Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split"
- [readme] Three hyperparameters for each of the CatBoost classifiers, one trained with balanced weights and one trained with equal weights, were tuned: n_estimators, depth and learning_rate, and the Bayesian hyperparameter tuning ran over 100 iterations, optimizing for the F1 score: "the Bayesian hyperparameter tuning ran over 100 iterations, optimizing for the F1 score at each iteration."
- [readme] Name of the output model pkl file that will be saved to the data/models/ folder: "# $4 - Name of the output model pkl file that will be saved to the data/models/ folder"
- [discussion] PhosBoost presents a precision-recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
