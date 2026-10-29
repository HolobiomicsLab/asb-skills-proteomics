---
name: imbalanced-label-evaluation-metrics
description: Use when benchmarking binary classifiers on sparsely labeled positives
  (e.g., Ser/Thr/Tyr phosphosites from qPTMplants) where accuracy and AUROC alone
  are mis and per-class recall/AUPRC tradeoffs must be reported.
when_to_use_negative:
- When the positive class is not rare or class labels are balanced — accuracy or ROC-based
  metrics alone may suffice and AUPRC offers no advantage
- When the goal is per-residue classification accuracy on a single pooled S/T/Y test
  set rather than per-class recall/AUPRC comparison across classifiers
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: PhosBoost
  role: Stacking classifier (two CatBoost base classifiers plus 5-fold CV logistic
    regression metaclassifier) evaluated via recall/AUPRC; run via predict.sh to emit
    probability CSVs
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosphoLingo
  role: Comparison classifier run under the 'full' setting with ProtT5-XL-U50 embeddings
    on the same test sets
- name: DeepPhos
  role: Comparison classifier run with default settings on the same test sets
- name: CatBoost
  role: Gradient-boosting base classifier used within the PhosBoost stacking model
    being evaluated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:f13fe4831ee68ad822ba940bc34c07a4c5506184c3b02dc3fdb04310a1a50a34
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# imbalanced-label-evaluation-metrics

## Summary

Evaluation strategy for binary phosphorylation-site classifiers trained on severely imbalanced labels, using recall and AUPRC (rather than accuracy) per residue class. In the PhosBoost paper this is applied to the A. thaliana qPTMplants test sets for combined S/T and Tyr models against PhosphoLingo and DeepPhos.

## When to use

Use when benchmarking binary classifiers on sparsely labeled positives (e.g., Ser/Thr/Tyr phosphosites from qPTMplants) where accuracy and AUROC alone are misleading and per-class recall/AUPRC tradeoffs must be reported.

## When NOT to use

- When the positive class is not rare or class labels are balanced — accuracy or ROC-based metrics alone may suffice and AUPRC offers no advantage
- When the goal is per-residue classification accuracy on a single pooled S/T/Y test set rather than per-class recall/AUPRC comparison across classifiers

## Inputs

- Per-site predicted probability CSV files (one per classifier) in data/preds/
- Stratified A. thaliana qPTMplants test-set site label files for the combined S/T and Y models
- Trained model files (e.g., Arabidopsi.qPTMplants.ST.joblib.pkl, Arabidopsi.qPTMplants.Y.joblib.pkl)

## Outputs

- benchmark_metrics.csv compiling recall and AUPRC for PhosBoost, PhosphoLingo, and DeepPhos on S/T and Y test sets
- Retained per-site probability prediction CSVs for downstream F-beta analysis

## How to apply

Load per-site predicted probability CSVs from data/preds/ (emitted by PhosBoost predict.sh, PhosphoLingo v0.1.0 'full' setting with ProtT5-XL-U50 embeddings, and DeepPhos with defaults) alongside the stratified A. thaliana test-set site label files. Compute recall and area under the precision-recall curve (AUPRC) separately for the combined S/T test set and the Y test set — do not pool residue classes, since Tyr labels are far more imbalanced. Report all three metrics per classifier; treat AUROC as a secondary check (all three S/T classifiers scored 0.88 AUROC while differing sharply in recall). Explicitly record zero-prediction outcomes (PhosphoLingo and DeepPhos predicted no Tyr sites) as recall = 0 rather than excluding the class. Retain the per-site probability files for downstream F-beta analysis.

## Related tools

- **PhosBoost** (Stacking classifier (two CatBoost base classifiers plus 5-fold CV logistic regression metaclassifier) evaluated via recall/AUPRC; run via predict.sh to emit probability CSVs) — https://github.com/eporetsky/PhosBoost
- **PhosphoLingo** (Comparison classifier run under the 'full' setting with ProtT5-XL-U50 embeddings on the same test sets)
- **DeepPhos** (Comparison classifier run with default settings on the same test sets)
- **CatBoost** (Gradient-boosting base classifier used within the PhosBoost stacking model being evaluated)

## Examples

```
python -c "from sklearn.metrics import recall_score, average_precision_score; import pandas as pd; df=pd.read_csv('data/preds/preds.csv'); y=df['y']; p=df['prob']; print(recall_score(y,p>0.5), average_precision_score(y,p))"
```

## Evaluation signals

- S/T recall should match approximately .78 (PhosBoost), .56 (PhosphoLingo), .14 (DeepPhos)
- S/T AUPRC should match approximately .54 (PhosBoost), .56 (PhosphoLingo), .42 (DeepPhos)
- PhosBoost Tyr recall should be approximately .6, while PhosphoLingo and DeepPhos should produce zero Tyr site predictions (recall = 0)
- AUROC secondary check: all three S/T classifiers should show a similar score of approximately .88, confirming AUROC alone does not discriminate
- Per-site probability CSVs in data/preds/ must exist for all three classifiers on both S/T and Y test sets before metrics are computed

## Limitations

- PhosBoost presents a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos, so high recall is achieved at some precision cost
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives compared to false positives, possibly due to higher label imbalance
- Many phosphosites remain undetected by high-throughput phosphoproteomic approaches, limiting the reliability of negative labels used in recall/AUPRC computation
- No software version numbers for DIAMOND, CatBoost, PhosphoLingo, or DeepPhos are stated beyond PhosphoLingo v0.1.0, complicating exact reproduction

## Evidence

- [abstract] S/T recall and AUPRC benchmarks across the three classifiers: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,"
- [abstract] Tyr zero-prediction outcome recorded as recall 0: "PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6"
- [results] AUROC as a non-discriminating secondary metric: "all three classifiers had a similar area under the receiver operating characteristic (AUROC) score of .88"
- [discussion] Precision-recall tradeoff limitation of high-recall classifiers: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
- [readme] Stratified splitting preserves label frequency, making the test-set metrics reflect the imbalanced label distribution: "We use the train_test_split (shuffle=True, stratify=y) function from scikit-learn to split the data of each residue individually while maintaing the same label frequency in each split"
