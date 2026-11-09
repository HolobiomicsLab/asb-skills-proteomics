---
name: gradient-boosting-classifier-training
description: Use when you need to train binary phosphorylation-site classifiers from
  protein language model embeddings, prioritizing recall with a balanced/equal-weight
  stacking design.
when_to_use_negative:
- Your goal is maximum precision / fewest false positives — PhosBoost has a precision-recall
  tradeoff with more predicted false positives than PhosphoLingo or DeepPhos
- Input embeddings or labels have not yet been generated/validated (non-canonical
  or missing amino acids, unmatched phosphosite positions) — data preparation must
  come first
- You only have a small, highly imbalanced dataset for tyrosine-like label imbalance
  where stacking inference of positives is expected to be uninformative
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3510
tools:
- name: CatBoost
  role: Gradient-boosting tree ensemble used as the two base classifiers (balanced
    and equal class weights) in the stacking classifier
- name: ProtT5-XL-U50
  role: Pre-trained protein language model generating residue and protein-wise average
    embedding vectors used as classifier input features
  repo: https://github.com/agemagician/ProtTrans
- name: PhosBoost
  role: Stacking classifier framework combining the two CatBoost base classifiers
    with a logistic regression metaclassifier
  repo: https://github.com/eporetsky/PhosBoost
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:9c987521df9095c4dd6b5e4b75a701cea0e494fa0ec28d853c050faca3a21cea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/agemagician/ProtTrans
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
yes_rate: 1.0
n_claims: 11
---

# gradient-boosting-classifier-training

## Summary

Train PhosBoost-style gradient-boosting phosphorylation classifiers: Bayesian-tuned CatBoost base classifiers (balanced and equal class weights) on ProtT5-XL-U50 embeddings, stacked via a 5-fold cross-validated logistic regression metaclassifier.

## When to use

Use when you need to train binary phosphorylation-site classifiers from protein language model embeddings, prioritizing recall with a balanced/equal-weight stacking design.

## When NOT to use

- Your goal is maximum precision / fewest false positives — PhosBoost has a precision-recall tradeoff with more predicted false positives than PhosphoLingo or DeepPhos
- Input embeddings or labels have not yet been generated/validated (non-canonical or missing amino acids, unmatched phosphosite positions) — data preparation must come first
- You only have a small, highly imbalanced dataset for tyrosine-like label imbalance where stacking inference of positives is expected to be uninformative

## Inputs

- Protein FASTA file (e.g. Arabidopsis.qPTMplants.fasta)
- Positive phosphosite list (qPTMplants.sites)
- ProtT5-XL-U50 residue and protein-wise average embedding pickle files (data/embeddings/)
- Tuned hyperparameter files (data/params/)

## Outputs

- Trained PhosBoost stacking model files (e.g. data/models/*.joblib.pkl for ST and Y)
- Hyperparameter tuning logs in log/ subfolder
- Test-set performance metrics (AUPRC, AUROC, recall; stacked-vs-base comparison table phosboost_model_comparison.csv)

## How to apply

First embed all candidate S/T/Y sites with the ProtT5-XL-U50 pre-trained pLM using a modified embed_ProtT5.ipynb that outputs residue and protein-wise average embedding vectors as a compressed pickle. Label sites y=1/0 from the qPTMplants phosphosite list, then apply a stratified 60%-20%-20% train/validation/test split (scikit-learn train_test_split with shuffle=True, stratify=y), combining Ser and Thr into a single ST dataset while keeping Tyr separate. Bayesian-tune n_estimators (50-2000), depth (2-10), and learning_rate (0.05-0.5) separately for balanced-weight and equal-weight CatBoost classifiers per model, over 100 iterations optimizing F1. Train two CatBoost (v1.1.1) base classifiers per residue model — one with auto_class_weights='balanced', one with default equal weights — and stack them with a logistic regression metaclassifier trained via 5-fold cross-validated predictions; save the resulting ST and Y stacking models as .pkl files under data/models/.

## Related tools

- **CatBoost** (Gradient-boosting tree ensemble used as the two base classifiers (balanced and equal class weights) in the stacking classifier)
- **ProtT5-XL-U50** (Pre-trained protein language model generating residue and protein-wise average embedding vectors used as classifier input features) — https://github.com/agemagician/ProtTrans
- **PhosBoost** (Stacking classifier framework combining the two CatBoost base classifiers with a logistic regression metaclassifier) — https://github.com/eporetsky/PhosBoost

## Examples

```
sbatch train.sh name ST name name  # train ST PhosBoost stacking model from data/embeddings/ using data/params/ hyperparameters, saving to data/models/
```

## Evaluation signals

- BayesianOptimization (100 iterations) reaches a converged F1 score for each of the four tuning runs (ST/Y × balanced/equal)
- Trained ST model reproduces reported benchmark on A. thaliana qPTMplants test set: recall ~0.78 with AUPRC ~0.54 and AUROC ~0.88
- Trained Y model achieves recall ~0.6 for tyrosine where PhosphoLingo and DeepPhos predict none
- Stacked classifier outperforms independently trained base classifiers (balanced-weight AUPRC .48, equal-weight .53, stacked .54) with higher recall at all probability thresholds
- Six site files produced per split (train/validation/test × ST/Y) with matched label frequencies across splits

## Limitations

- PhosBoost presents a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, possibly due to higher label imbalance
- The mechanism behind the stacking benefit (balanced class weights addressing class imbalance) is a stated hypothesis, not demonstrated

## Evidence

- [results] Two CatBoost base classifiers and 5-fold CV metaclassifier: "The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency"
- [results] Metaclassifier training approach: "the logistic regression metaclassifier that was trained using a 5‐fold cross‐validated prediction approach"
- [results] Split procedure: "Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split"
- [abstract] Benchmark performance: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,"
- [readme] Embedding generation and pickle output: [evidence span withheld — anchor status: not_found]
- [readme] Hyperparameter tuning detail: "Three hyperparameters for each of the CatBoost classifiers, one trained with balanced weights and one trained with equal weights, were tuned: n_estimators, depth and learning_rate"
