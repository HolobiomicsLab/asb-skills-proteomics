---
name: class-imbalance-weighted-training
description: Use when training binary classifiers on datasets with strong positive/negative
  label imbalance (e.g., rare phosphorylation sites in plants) where recall on the
  minority class is the priority and standard training under-predicts positives.
when_to_use_negative:
- The dataset is already balanced or you are optimizing precision/AUPRC rather than
  recall — the article notes PhosBoost has a precision-recall tradeoff with increased
  predicted false positives compared to PhosphoLingo and DeepPhos.
- You only need a single learner and cannot afford per-classifier hyperparameter tuning
  — the balanced-weight base classifier alone scored lower AUPRC (.48) than the equal-weight
  one (.53) on the A. thaliana test set.
- The input is raw sequence without precomputed protein language model embeddings
  — this skill presupposes ProtT5 residue and protein-average embedding features.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3510
tools:
- name: CatBoost
  role: Gradient-boosting tree base classifiers; one instance trained with auto_class_weights='balanced',
    one with equal class weights
- name: PhosBoost
  role: Stacking classifier framework that combines the two weighted CatBoost base
    classifiers with a logistic regression metaclassifier
  repo: https://github.com/eporetsky/PhosBoost
- name: scikit-learn
  role: Provides the logistic regression metaclassifier (balanced class weights),
    5-fold cross-validated stacking, and train_test_split with stratify=y
- name: BayesianOptimization
  role: Bayesian hyperparameter tuning (100 iterations, F1 objective) performed separately
    for the balanced-weight and equal-weight classifiers
- name: ProtT5-XL-U50
  role: Pre-trained protein language model generating residue and protein-wise average
    embedding vectors used as classifier features
  repo: https://github.com/agemagician/ProtTrans
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:a48f8bb64d7d275e2f9335d905a493978b4168d1cc57727f75b89dc1f245f4d0
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
yes_rate: 0.9411764705882353
n_claims: 17
---

# class-imbalance-weighted-training

## Summary

Train a balanced-weight classifier alongside an equal-weight classifier to handle severe class imbalance in phosphorylation prediction, then stack both with a logistic regression metaclassifier. PhosBoost uses this dual-CatBoost design because tyrosine phosphosites have far fewer positives than negatives, hurting recall.

## When to use

Use when training binary classifiers on datasets with strong positive/negative label imbalance (e.g., rare phosphorylation sites in plants) where recall on the minority class is the priority and standard training under-predicts positives.

## When NOT to use

- The dataset is already balanced or you are optimizing precision/AUPRC rather than recall — the article notes PhosBoost has a precision-recall tradeoff with increased predicted false positives compared to PhosphoLingo and DeepPhos.
- You only need a single learner and cannot afford per-classifier hyperparameter tuning — the balanced-weight base classifier alone scored lower AUPRC (.48) than the equal-weight one (.53) on the A. thaliana test set.
- The input is raw sequence without precomputed protein language model embeddings — this skill presupposes ProtT5 residue and protein-average embedding features.

## Inputs

- Labeled ProtT5-XL-U50 embedding vectors for S/T/Y sites plus protein-wise average embeddings (compressed pickle file) with a 'y' label column (1=positive phosphosite, 0=negative)
- Stratified 60%-20%-20% train/validation/test split site-list files for the ST and Y residue models

## Outputs

- Two tuned CatBoost base classifiers per residue model (balanced-weight and equal-weight)
- Stacked PhosBoost model (.pkl/joblib file) combining both base classifiers with a logistic regression metaclassifier
- Comparison metrics (AUPRC, recall, F1) of stacked vs. independently trained base classifiers

## How to apply

For each residue model (combined Ser/Thr 'ST' and separate Tyr 'Y'), train two CatBoost v1.1.1 gradient-boosting tree base classifiers on ProtT5-XL-U50 residue embeddings plus protein-wise average embeddings: one with auto_class_weights='balanced' (weights based on label frequency) and one with default equal class weights. Tune each classifier's hyperparameters separately via BayesianOptimization over 100 iterations, optimizing F1, with n_estimators in 50–2000, depth in 2–10, and learning_rate in 0.05–0.5. Stack the two base classifiers' predicted probabilities into a scikit-learn logistic regression metaclassifier (balanced class weights) trained on 5-fold cross-validated predictions. Rationale: the article hypothesizes balanced class weights help address class imbalance, and the ablation showed the balanced-weight classifier alone had lower AUPRC (.48) than the equal-weight one (.53) or the stacking classifier (.54), with PhosBoost achieving higher recall at all thresholds.

## Related tools

- **CatBoost** (Gradient-boosting tree base classifiers; one instance trained with auto_class_weights='balanced', one with equal class weights)
- **PhosBoost** (Stacking classifier framework that combines the two weighted CatBoost base classifiers with a logistic regression metaclassifier) — https://github.com/eporetsky/PhosBoost
- **scikit-learn** (Provides the logistic regression metaclassifier (balanced class weights), 5-fold cross-validated stacking, and train_test_split with stratify=y)
- **BayesianOptimization** (Bayesian hyperparameter tuning (100 iterations, F1 objective) performed separately for the balanced-weight and equal-weight classifiers)
- **ProtT5-XL-U50** (Pre-trained protein language model generating residue and protein-wise average embedding vectors used as classifier features) — https://github.com/agemagician/ProtTrans

## Examples

```
sbatch tune.sh name ST 100 balanced; sbatch tune.sh name Y 100 equal; sbatch train.sh name ST params params
```

## Evaluation signals

- Hyperparameter tuning converges on F1 score after 100 Bayesian optimization iterations for each of the four tuned classifiers (ST-balanced, ST-equal, Y-balanced, Y-equal).
- On the A. thaliana qPTMplants test set, the stacked model outperforms both independent base classifiers: AUPRC .54 vs .53 (equal weights) and .48 (balanced weights), with higher recall at all probability thresholds.
- ST model reaches recall ~0.78 with AUPRC ~0.54; Y model reaches recall ~0.6 where PhosphoLingo and DeepPhos predicted no tyrosine sites.
- All three S/T classifiers show a similar AUROC of ~0.88, so improvements should appear in AUPRC/recall rather than AUROC.
- Saved model files appear under data/models/ as .pkl joblib files for the ST and Y models.

## Limitations

- The benefit of balanced class weights in the stacking classifier is stated as a hypothesis ('We hypothesize that the use of balanced class weights to train one base classifier helps address the class imbalance'), not demonstrated mechanistically.
- PhosBoost shows a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos.
- For Tyr phosphosites, the model was less informative at predicting inferred positives, possibly due to higher label imbalance; performance improvements from larger data remain unvalidated hypotheses.
- Negative labels are unreliable because many phosphosites remain undetected by high-throughput phosphoproteomic approaches.

## Evidence

- [results] CatBoost base classifiers with balanced vs equal class weights, stacked with logistic regression: "The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency"
- [results] 5-fold cross-validated logistic regression metaclassifier: "the logistic regression metaclassifier that was trained using a 5‐fold cross‐validated prediction approach"
- [results] Ablation: balanced-weight base classifier underperforms the stacked classifier: [evidence span withheld — anchor status: not_found]
- [readme] Bayesian tuning setup per classifier: "Three hyperparameters for each of the CatBoost classifiers, one trained with balanced weights and one trained with equal weights, were tuned: n_estimators, depth and learning_rate"
- [discussion] Balanced-class-weight rationale stated as hypothesis: "We hypothesize that the use of balanced class weights to train one base classifier helps address the class imbalance"
- [intro] Y Tyr label imbalance motivates the weighted approach: "the prediction of Tyr phosphorylation poses additional challenges, partially due to a smaller amount of experimental data leading to a relatively higher label imbalance"
