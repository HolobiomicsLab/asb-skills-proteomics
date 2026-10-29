---
name: classification-threshold-beta-sweep
description: Use when comparing phosphosite classifiers whose AUROC or AUPRC alone
  masks recall differences, and the analysis goal is to quantify at what recall-vs-precision
  weighting (beta) one classifier overtakes another.
when_to_use_negative:
- When the question is overall ranking by area under the precision-recall curve —
  all three S/T classifiers have competitive AUPRC (.54, .56, .42) and tied AUROC
  (.88), so the beta sweep adds value only for recall-weighted comparison
- When evaluating tyrosine models where PhosphoLingo and DeepPhos failed to predict
  any tyrosine sites — their F-beta curves are degenerate, making the sweep uninformative
  for those competitors
tools:
- name: PhosBoost
  role: Classifier under evaluation; expected to surpass competitors in F-beta at
    beta ≈ 2
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosphoLingo
  role: Competitor classifier scored across the beta sweep
- name: DeepPhos
  role: Competitor classifier scored across the beta sweep
- name: scikit-learn
  role: Provides fbeta_score and train_test_split (shuffle=True, stratify=y) used
    to generate the ground-truth test split
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:dd62408686694d31594136de273b31eb8e374ace6725346d3b6c9a2f5e145b0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DeepPhos
  - PhosBoost
  - PhosphoLingo
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# classification-threshold-beta-sweep

## Summary

Sweeps the F-beta score over a grid of beta values (e.g. 0.25–3) to compare classifiers under different precision–recall weightings, identifying the beta at which a recall-oriented classifier (PhosBoost) surpasses competitors on the A. thaliana qPTMplants benchmark (Figure 3d).

## When to use

Use when comparing phosphosite classifiers whose AUROC or AUPRC alone masks recall differences, and the analysis goal is to quantify at what recall-vs-precision weighting (beta) one classifier overtakes another.

## When NOT to use

- When the question is overall ranking by area under the precision-recall curve — all three S/T classifiers have competitive AUPRC (.54, .56, .42) and tied AUROC (.88), so the beta sweep adds value only for recall-weighted comparison
- When evaluating tyrosine models where PhosphoLingo and DeepPhos failed to predict any tyrosine sites — their F-beta curves are degenerate, making the sweep uninformative for those competitors

## Inputs

- Per-site predicted probability CSV files (data/preds/) for PhosBoost, PhosphoLingo, and DeepPhos, for both S/T and Y models
- Held-out test-set site labels: the 20% stratified test split of the A. thaliana qPTMplants Ser/Thr/Tyr phosphosite data (60%-20%-20% train/validation/test split)
- Grid of beta values (e.g. 0.25 to 3)

## Outputs

- Beta-sweep score table (fbeta_scores.csv) with precision, recall, and F-beta per classifier per beta
- F-beta versus beta plot for all three classifiers (Figure 3d panel, S/T and Y models)

## How to apply

Load per-site predicted probability CSVs from the benchmark rerun (data/preds/) for PhosBoost, PhosphoLingo, and DeepPhos for the S/T model (and separately the Y model), plus the held-out 20% stratified test-split labels of the A. thaliana qPTMplants data as ground truth. Compute precision, recall, and F-beta with scikit-learn's fbeta_score, sweeping beta over a grid from ~0.25 to 3, per model (ST vs Y analyzed separately). Identify where PhosBoost's F-beta curve crosses above PhosphoLingo and DeepPhos (expected near beta ≈ 2, i.e. recall weighted roughly twice as heavily as precision). Plot F-beta versus beta for all three classifiers to reproduce the Figure 3d panel, and write the full beta-sweep score table to fbeta_scores.csv. The rationale is that AUROC is tied (~.88 for all three S/T classifiers), so only a recall-weighted metric reveals PhosBoost's advantage (ST recall .78 vs .56 and .14).

## Related tools

- **PhosBoost** (Classifier under evaluation; expected to surpass competitors in F-beta at beta ≈ 2) — https://github.com/eporetsky/PhosBoost
- **PhosphoLingo** (Competitor classifier scored across the beta sweep)
- **DeepPhos** (Competitor classifier scored across the beta sweep)
- **scikit-learn** (Provides fbeta_score and train_test_split (shuffle=True, stratify=y) used to generate the ground-truth test split)

## Examples

```
from sklearn.metrics import fbeta_score; fb = fbeta_score(y_true_test, y_pred, beta=2)
```

## Evaluation signals

- Regenerated F-beta curves must show PhosBoost surpassing both PhosphoLingo and DeepPhos near beta ≈ 2, reproducing the Figure 3d pattern
- Underlying test-set metrics must be consistent with reported ST recall of .78 (PhosBoost), .56 (PhosphoLingo), and .14 (DeepPhos) with AUPRC .54, .56, .42
- fbeta_scores.csv must contain the full beta-sweep table covering the grid (e.g. beta 0.25–3) for both S/T and Y models
- All three S/T classifiers should score similarly at low beta (precision-weighted), converging near their AUPRC values, before diverging as beta increases

## Limitations

- PhosBoost presents a precision-recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos, so its F-beta advantage appears only when recall is weighted more heavily
- For tyrosine, PhosphoLingo and DeepPhos predicted no sites (recall 0), so F-beta comparison for the Y model is effectively one-sided
- Label uncertainty and mislabeling in qPTMplants data limits the reliability of negative labels used in the scoring

## Evidence

- [abstract] PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively): "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve"
- [results] All three S/T classifiers tied on AUROC: "all three classifiers had a similar area under the receiver operating characteristic (AUROC) score of .88"
- [readme] Recall prioritization rationale: "PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores."
- [methods] Stratified split used to generate ground-truth test labels: "randomly split into stratified training, validation, and testing sets using a 60%–20%–20%, split, respectively"
