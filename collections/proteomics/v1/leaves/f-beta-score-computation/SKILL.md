---
name: f-beta-score-computation
description: Use when comparing classifiers on imbalanced phosphosite predictions
  and you need to know which method wins when recall is prioritized over precision
  (beta > 1), not just at F1 or AUPRC.
when_to_use_negative:
- When the question is overall ranking by area under the precision-recall curve (AUPRC)
  rather than recall-prioritized performance — the article's AUPRC comparison (.54,
  .56, .42) is a different analysis
- When predictions are already hard labels and you need probability-threshold-free
  metrics like AUROC (all three S/T classifiers scored .88 AUROC, so AUROC cannot
  separate them here)
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scikit-learn
  role: fbeta_score function used to compute precision, recall, and F-beta over the
    beta grid; train_test_split (shuffle=True, stratify=y) generated the labeled test
    split
- name: PhosBoost
  role: Classifier under evaluation; expected to surpass competitors in F-beta near
    beta = 2 due to higher recall
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosphoLingo
  role: Comparator classifier included in the beta sweep
- name: DeepPhos
  role: Comparator classifier included in the beta sweep
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:aac6c826a511c0934afdb2dd44360245f3cf30add630641b554000ba15eee188
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
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# f-beta-score-computation

## Summary

Sweep the F-beta score over a grid of beta values (e.g. 0.25–3) using scikit-learn's fbeta_score to evaluate how phosphosite classifiers (PhosBoost, PhosphoLingo, DeepPhos) behave when recall is weighted more heavily than precision, reproducing the Figure 3d panel of the PhosBoost paper.

## When to use

Use when comparing classifiers on imbalanced phosphosite predictions and you need to know which method wins when recall is prioritized over precision (beta > 1), not just at F1 or AUPRC.

## When NOT to use

- When the question is overall ranking by area under the precision-recall curve (AUPRC) rather than recall-prioritized performance — the article's AUPRC comparison (.54, .56, .42) is a different analysis
- When predictions are already hard labels and you need probability-threshold-free metrics like AUROC (all three S/T classifiers scored .88 AUROC, so AUROC cannot separate them here)

## Inputs

- Per-site predicted probability CSV files per classifier (data/preds/) for PhosBoost, PhosphoLingo, and DeepPhos, S/T and Y models
- Held-out test-set site labels (20% stratified test split of A. thaliana qPTMplants sites) as ground truth
- Beta grid spanning approximately 0.25 to 3

## Outputs

- F-beta versus beta curves per classifier (Figure 3d reproduction) for S/T and Y models
- fbeta_scores.csv containing the full beta-sweep score table (precision, recall, F-beta per classifier per beta)

## How to apply

Load per-site predicted probabilities from the benchmark rerun (data/preds/) for PhosBoost, PhosphoLingo, and DeepPhos for the S/T model and separately the Y model, plus the held-out test-set labels from the 60%-20%-20% stratified split of the A. thaliana qPTMplants data. Threshold the probabilities to get binary predictions, then call scikit-learn's fbeta_score with beta swept over a grid from roughly 0.25 to 3, computing precision, recall, and F-beta per classifier per model. Identify the beta at which PhosBoost's curve crosses above the competitors — expected near beta = 2, where recall is weighted about twice as much as precision. Plot F-beta versus beta for all three classifiers (Figure 3d) and export the full score table to fbeta_scores.csv.

## Related tools

- **scikit-learn** (fbeta_score function used to compute precision, recall, and F-beta over the beta grid; train_test_split (shuffle=True, stratify=y) generated the labeled test split)
- **PhosBoost** (Classifier under evaluation; expected to surpass competitors in F-beta near beta = 2 due to higher recall) — https://github.com/eporetsky/PhosBoost
- **PhosphoLingo** (Comparator classifier included in the beta sweep)
- **DeepPhos** (Comparator classifier included in the beta sweep)

## Examples

```
from sklearn.metrics import fbeta_score; fb = fbeta_score(y_true, y_pred, beta=2)
```

## Evaluation signals

- The regenerated F-beta curves must show PhosBoost crossing above both PhosphoLingo and DeepPhos near beta = 2 (Figure 3d pattern)
- At beta = 1 (F1), relative orderings should remain consistent with the reported precision/recall: PhosBoost S/T recall .78 vs .56 (PhosphoLingo) and .14 (DeepPhos)
- fbeta_scores.csv must contain one row per (classifier, beta, model) combination covering the full 0.25–3 beta grid
- Curves should be computed separately for the S/T model and the Y model, and the Y analysis should reflect that PhosphoLingo and DeepPhos predicted no tyrosine sites while PhosBoost reached recall .6

## Limitations

- The F-beta advantage reflects PhosBoost's known precision-recall tradeoff: it produces more predicted false positives than PhosphoLingo and DeepPhos, so the win at beta ~ 2 assumes recall genuinely matters more for the downstream use case
- Scores depend on the specific 20% stratified test split; label uncertainty in qPTMplants-derived negative labels (undetected phosphosites) partially addressed only by the DIAMOND inferred-positive relabeling step
- For tyrosine, results are less informative: PhosphoLingo and DeepPhos failed to predict any Y sites, so F-beta comparisons on the Y model are degenerate for those classifiers

## Evidence

- [abstract] PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,"
- [abstract] PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6: "PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6"
- [discussion] PhosBoost presents a precision-recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos: [evidence span withheld — anchor status: not_found]
- [results] Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%-20%-20% split: "Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split"
- [readme] PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores: "PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores."
