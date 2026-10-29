---
name: precision-recall-metric-computation
description: Use when comparing phosphorylation-site classifiers on imbalanced positive/negative
  site labels and you need recall and AUPRC (rather than accuracy) from per-site predicted
  probability outputs.
when_to_use_negative:
- When the goal is ranking-independent threshold selection rather than comparing classifiers
  — AUROC alone was identical (.88) across all three S/T classifiers and will not
  discriminate
- When the evaluation set is not imbalanced (phosphosite labels are highly imbalanced
  in qPTMplants); accuracy-style metrics may suffice and PR curves add little
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3510
tools:
- name: PhosBoost
  role: Stacking classifier (two CatBoost base classifiers with balanced/equal class
    weights + 5-fold CV logistic regression metaclassifier) whose predictions are
    scored
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosphoLingo
  role: Comparison classifier run under the 'full' setting with pre-trained ProtT5-XL-U50
    embeddings
- name: DeepPhos
  role: Comparison classifier run with default settings on the same test sets
- name: ProtT5-XL-U50
  role: Pre-trained protein language model used to generate residue and protein-average
    embeddings for prediction inputs
  repo: https://github.com/agemagician/ProtTrans
- name: CatBoost
  role: Gradient-boosting tree base classifiers inside the PhosBoost stacking model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:305b2be12dd497194c33492940cf4fe7d4886634ebec5daf6dfa60733f17b94d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - CatBoost
  - DeepPhos
  - PhosBoost
  - PhosphoLingo
  - ProtT5-XL-U50
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# precision-recall-metric-computation

## Summary

Compute recall and area under the precision-recall curve (AUPRC) for phosphorylation-site classifiers such as PhosBoost, PhosphoLingo, and DeepPhos on the A. thaliana qPTMplants S/T and Y test sets, emphasizing recall under class imbalance.

## When to use

Use when comparing phosphorylation-site classifiers on imbalanced positive/negative site labels and you need recall and AUPRC (rather than accuracy) from per-site predicted probability outputs.

## When NOT to use

- When the goal is ranking-independent threshold selection rather than comparing classifiers — AUROC alone was identical (.88) across all three S/T classifiers and will not discriminate
- When the evaluation set is not imbalanced (phosphosite labels are highly imbalanced in qPTMplants); accuracy-style metrics may suffice and PR curves add little

## Inputs

- Per-site predicted probability CSV files in data/preds/ from each classifier
- Stratified A. thaliana qPTMplants test-set site label files with 'y' column (positive=1, negative=0) for S/T and Y models
- Trained PhosBoost model pickles (Arabidopsi.qPTMplants.ST.joblib.pkl, Arabidopsi.qPTMplants.Y.joblib.pkl)

## Outputs

- benchmark_metrics.csv with recall and AUPRC per classifier (PhosBoost, PhosphoLingo, DeepPhos) for S/T and Y
- Per-site probability prediction CSV files retained for downstream F-beta analysis

## How to apply

Run each classifier (PhosBoost via predict.sh with its CatBoost stacking models, PhosphoLingo v0.1.0 'full' setting with ProtT5-XL-U50 embeddings, DeepPhos with defaults) on the same stratified 20% test split of the qPTMplants embedding/site data, emitting per-site predicted probability CSVs. Treat the 'y' label column from the sites files as ground truth and compute recall (fraction of true phosphosites recovered) and AUPRC separately for the combined S/T model and the Y model. Expect AUPRC, not accuracy, to discriminate classifiers under heavy label imbalance; also record AUROC (all three S/T classifiers scored .88) as a secondary check. Compile per-classifier, per-residue-class metrics into benchmark_metrics.csv and retain the per-site probability CSVs for downstream F-beta analysis.

## Related tools

- **PhosBoost** (Stacking classifier (two CatBoost base classifiers with balanced/equal class weights + 5-fold CV logistic regression metaclassifier) whose predictions are scored) — https://github.com/eporetsky/PhosBoost
- **PhosphoLingo** (Comparison classifier run under the 'full' setting with pre-trained ProtT5-XL-U50 embeddings)
- **DeepPhos** (Comparison classifier run with default settings on the same test sets)
- **ProtT5-XL-U50** (Pre-trained protein language model used to generate residue and protein-average embeddings for prediction inputs) — https://github.com/agemagician/ProtTrans
- **CatBoost** (Gradient-boosting tree base classifiers inside the PhosBoost stacking model)

## Examples

```
sbatch predict.sh name ST; sbatch predict.sh name Y  # emit per-site probability CSVs to data/preds/, then compute recall/AUPRC for PhosBoost, PhosphoLingo, and DeepPhos on the shared test split
```

## Evaluation signals

- S/T recall should approximate .78/.56/.14 for PhosBoost/PhosphoLingo/DeepPhos
- S/T AUPRC should approximate .54/.56/.42 for PhosBoost/PhosphoLingo/DeepPhos
- PhosBoost Tyr recall should approximate .6, while PhosphoLingo and DeepPhos produce zero Tyr site predictions
- All three S/T classifiers should show a similar AUROC of .88 on the A. thaliana qPTMplants test set
- Per-site probability CSVs for all classifiers must be present in data/preds/ before compiling benchmark_metrics.csv

## Limitations

- PhosBoost presents a precision-recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, possibly due to higher label imbalance
- Many phosphosites remain undetected by high-throughput phosphoproteomic approaches, limiting negative label reliability and thus metric interpretation

## Evidence

- [abstract] PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,"
- [abstract] PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6: "PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6"
- [results] Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split: "Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split"
- [results] all three classifiers had a similar area under the receiver operating characteristic (AUROC) score of .88: "all three classifiers had a similar area under the receiver operating characteristic (AUROC) score of .88"
- [discussion] PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
