---
name: classifier-performance-curve-visualization
description: Use when comparing classifiers whose AUPRC/AUROC are similar but whose
  recall priorities differ, and you need to identify at which beta (recall-weighting)
  one classifier's F-beta surpasses another.
when_to_use_negative:
- When the question is overall ranking by precision-recall performance alone (AUPRC
  comparison), not recall-weighted tradeoffs — PhosBoost and PhosphoLingo had comparable
  AUPRC (.54 vs .56) so F-beta sweep is the discriminating view
- When predictions are hard labels rather than per-site probabilities scored against
  the same stratified test split, since fbeta_score at each beta requires consistent
  predictions and ground-truth labels
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0601
tools:
- name: PhosBoost
  role: Classifier under evaluation whose recall-oriented F-beta advantage at beta
    ~ 2 is being demonstrated
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosphoLingo
  role: Comparator classifier scored on the same test split across the beta sweep
- name: DeepPhos
  role: Comparator classifier scored on the same test split across the beta sweep
- name: scikit-learn
  role: Provides fbeta_score and train_test_split (stratify=y) used for the beta sweep
    and the underlying 60%-20%-20% split
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:a7128aa9e30de548ca4d744beefc75d81e5ebfdb268cef7f9dc4d741652b2e91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - DeepPhos
  - PhosBoost
  - PhosphoLingo
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# classifier-performance-curve-visualization

## Summary

Computes and plots F-beta scores over a sweep of beta values to expose the precision-recall tradeoff between phosphorylation-site classifiers (PhosBoost, PhosphoLingo, DeepPhos) on the A. thaliana qPTMplants benchmark, reproducing the Figure 3d analysis.

## When to use

Use when comparing classifiers whose AUPRC/AUROC are similar but whose recall priorities differ, and you need to identify at which beta (recall-weighting) one classifier's F-beta surpasses another.

## When NOT to use

- When the question is overall ranking by precision-recall performance alone (AUPRC comparison), not recall-weighted tradeoffs — PhosBoost and PhosphoLingo had comparable AUPRC (.54 vs .56) so F-beta sweep is the discriminating view
- When predictions are hard labels rather than per-site probabilities scored against the same stratified test split, since fbeta_score at each beta requires consistent predictions and ground-truth labels

## Inputs

- Per-site predicted probability CSV files from data/preds/ for PhosBoost, PhosphoLingo, and DeepPhos (S/T and Y models)
- Held-out test-set site labels (20% stratified test split of A. thaliana qPTMplants Ser/Thr/Tyr data)

## Outputs

- F-beta versus beta curves per classifier and model (Figure 3d panel)
- fbeta_scores.csv containing the full beta-sweep precision, recall, and F-beta score table
- Identified crossover beta value (~2) where PhosBoost outperforms competitors

## How to apply

Load per-site predicted probability CSVs from the benchmark rerun (data/preds/) for PhosBoost, PhosphoLingo, and DeepPhos for the S/T and Y models, and the held-out 20% stratified test-split site labels from the 60%-20%-20% A. thaliana qPTMplants split as ground truth. Using scikit-learn's fbeta_score, sweep beta over a grid spanning roughly 0.25 to 3, computing precision, recall, and F-beta per classifier per model (S/T separately from Y). Identify the beta where PhosBoost's curve crosses above the competitors (expected near beta = 2, where recall is weighted about twice as much as precision). Plot F-beta versus beta for all three classifiers per model to reproduce Figure 3d, and write the full score table to fbeta_scores.csv. Validation hinges on reproducing the reported pattern: PhosBoost's S/T recall (.78) exceeds PhosphoLingo (.56) and DeepPhos (.14), so its F-beta curve should dominate at higher beta.

## Related tools

- **PhosBoost** (Classifier under evaluation whose recall-oriented F-beta advantage at beta ~ 2 is being demonstrated) — https://github.com/eporetsky/PhosBoost
- **PhosphoLingo** (Comparator classifier scored on the same test split across the beta sweep)
- **DeepPhos** (Comparator classifier scored on the same test split across the beta sweep)
- **scikit-learn** (Provides fbeta_score and train_test_split (stratify=y) used for the beta sweep and the underlying 60%-20%-20% split)

## Examples

```
from sklearn.metrics import fbeta_score; f2 = fbeta_score(y_test, (preds >= threshold).astype(int), beta=2)  # sweep beta 0.25–3 per classifier, write fbeta_scores.csv
```

## Evaluation signals

- Regenerated F-beta curves reproduce Figure 3d: PhosBoost surpasses PhosphoLingo and DeepPhos at beta approximately 2
- Computed S/T recall values match reported values: PhosBoost .78, PhosphoLingo .56, DeepPhos .14
- AUPRC values match reported values: PhosBoost .54, PhosphoLingo .56, DeepPhos .42
- fbeta_scores.csv contains one row per (classifier, model, beta) covering the swept beta grid (e.g. 0.25 to 3)
- S/T and Y models are scored separately; Y results should reflect PhosphoLingo and DeepPhos failing to predict any Tyr sites (recall 0) vs PhosBoost recall .6

## Limitations

- F-beta at high beta favors PhosBoost because it trades precision for recall; PhosBoost presents a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- Negative labels are unreliable since many phosphosites remain undetected by high-throughput phosphoproteomic approaches, which distorts precision estimates
- PhosBoost's F-beta advantage reflects predictions before DIAMOND inferred-positive relabeling; the alignment step improved results for all classifiers and should be kept consistent across comparisons

## Evidence

- [abstract] PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,"
- [results] Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split: "Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split"
- [discussion] PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
- [readme] PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores.: "PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores."
- [readme] We use the train_test_split (shuffle=True, stratify=y) function from scikit-learn to split the data of each residue individually: "We use the train_test_split (shuffle=True, stratify=y) function from scikit-learn to split the data of each residue individually"
