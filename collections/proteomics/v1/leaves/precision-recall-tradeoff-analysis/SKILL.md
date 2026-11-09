---
name: precision-recall-tradeoff-analysis
description: Use when comparing classifiers on imbalanced phosphosite prediction data
  where recall matters more than precision, e.g. selecting a method whose F-beta curve
  crosses above competitors near beta ≈ 2.
when_to_use_negative:
- The goal is balanced F1 or AUPRC-based model selection rather than recall-prioritized
  selection — the article notes all three S/T classifiers have similar AUROC (.88)
  and PhosBoost's advantage only appears at higher beta
- Labels are unreliable negatives (undetected phosphosites from high-throughput runs),
  since precision/FP counts are inflated by label uncertainty that the DIAMOND inferred-positives
  step only partially addresses
- The input is already aggregated metric scores rather than per-site predictions plus
  test labels — F-beta cannot be recomputed without site-level predictions and ground
  truth
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0601
tools:
- name: PhosBoost
  role: Candidate classifier whose per-site probabilities are scored; expected to
    surpass competitors in F-beta at beta ≈ 2
  repo: https://github.com/eporetsky/PhosBoost
- name: PhosphoLingo
  role: Comparator classifier with ST recall .56; its F-beta curve is compared against
    PhosBoost's across beta
- name: DeepPhos
  role: Comparator classifier with ST recall .14; its F-beta curve is compared against
    PhosBoost's across beta
- name: scikit-learn
  role: Provides fbeta_score and the train_test_split (shuffle=True, stratify=y) used
    to generate the 20% test split that supplies ground-truth labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:c64f2ee4aed057a0731606cc06a3c0c61fa50b3bec9137232adea1d3b5a75262
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# precision-recall-tradeoff-analysis

## Summary

Quantifies how phosphorylation-site classifiers compare when recall is weighted more heavily than precision, by sweeping the beta parameter of the F-beta score (precision-, recall-, and beta-swept via scikit-learn) across candidate classifiers and plotting F-beta versus beta.

## When to use

Use when comparing classifiers on imbalanced phosphosite prediction data where recall matters more than precision, e.g. selecting a method whose F-beta curve crosses above competitors near beta ≈ 2.

## When NOT to use

- The goal is balanced F1 or AUPRC-based model selection rather than recall-prioritized selection — the article notes all three S/T classifiers have similar AUROC (.88) and PhosBoost's advantage only appears at higher beta
- Labels are unreliable negatives (undetected phosphosites from high-throughput runs), since precision/FP counts are inflated by label uncertainty that the DIAMOND inferred-positives step only partially addresses
- The input is already aggregated metric scores rather than per-site predictions plus test labels — F-beta cannot be recomputed without site-level predictions and ground truth

## Inputs

- Per-site predicted probability CSV files from the benchmark rerun (data/preds/) for PhosBoost, PhosphoLingo, and DeepPhos (S/T and Y models)
- Held-out test-set site labels from the 20% stratified test split of the A. thaliana qPTMplants data (60%-20%-20% train/validation/test split)

## Outputs

- F-beta versus beta curves/plot for all three classifiers per model (Figure 3d panel reproduction)
- Beta-sweep score table written to fbeta_scores.csv (precision, recall, F-beta per classifier per beta)

## How to apply

Load per-site predicted probability outputs from the candidate classifiers (e.g. PhosBoost, PhosphoLingo, DeepPhos S/T and Y models on A. thaliana qPTMplants data) together with the held-out 20% stratified test-split site labels as ground truth. Using scikit-learn's fbeta_score, compute precision, recall, and F-beta for each classifier while sweeping beta over a grid (e.g. 0.25 to 3), for the S/T model and the Y model separately. Identify the beta at which one classifier's F-beta curve crosses above the others (for PhosBoost this is near beta = 2, where recall is weighted twice as much as precision) and plot F-beta versus beta for all classifiers, reproducing the Figure 3d panel. Write the full beta-sweep score table to fbeta_scores.csv. The analysis is motivated by PhosBoost's recall-oriented advantage: ST recall .78 versus .56 (PhosphoLingo) and .14 (DeepPhos), despite PhosBoost having more predicted false positives.

## Related tools

- **PhosBoost** (Candidate classifier whose per-site probabilities are scored; expected to surpass competitors in F-beta at beta ≈ 2) — https://github.com/eporetsky/PhosBoost
- **PhosphoLingo** (Comparator classifier with ST recall .56; its F-beta curve is compared against PhosBoost's across beta)
- **DeepPhos** (Comparator classifier with ST recall .14; its F-beta curve is compared against PhosBoost's across beta)
- **scikit-learn** (Provides fbeta_score and the train_test_split (shuffle=True, stratify=y) used to generate the 20% test split that supplies ground-truth labels)

## Examples

```
from sklearn.metrics import fbeta_score; for b in [0.25,0.5,1,2,3]: print(b, fbeta_score(y_true, y_pred, beta=b))
```

## Evaluation signals

- The regenerated F-beta curves must show PhosBoost surpassing both PhosphoLingo and DeepPhos at beta approximately 2 (Figure 3d pattern)
- ST recall values at the operating threshold must match the reported values: PhosBoost .78, PhosphoLingo .56, DeepPhos .14
- AUPRC values must fall in the reported competitive range (.54, .56, .42) confirming that PhosBoost's advantage is recall-driven, not precision-driven
- fbeta_scores.csv must contain a full sweep grid (e.g. beta 0.25–3) with precision, recall, and F-beta per classifier for both S/T and Y models
- Test-set labels must come from the same 60%-20%-20% stratified split described in the methods; mismatched splits will not reproduce the crossing point

## Limitations

- PhosBoost presents a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos, so high-beta superiority comes at a precision cost
- For Tyr phosphosites, PhosphoLingo and DeepPhos predicted no sites at all, making Y-model comparisons degenerate (PhosBoost Y recall .6)
- Label uncertainty and mislabeling are common and only partially addressed by DIAMOND relabeling of inferred positives, which can bias F-beta estimates

## Evidence

- [abstract] PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve (.54, .56, and .42,"
- [results] Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split: "Random stratification was used on the individual Ser, Thr, and Tyr datasets to generate the training, validation, and test sets using a 60%–20%–20% split"
- [discussion] PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
- [readme] PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores: "PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores."
- [readme] we use the train_test_split (shuffle=True, stratify=y) function from scikit-learn to split the data of each residue individually: "We use the train_test_split (shuffle=True, stratify=y) function from scikit-learn to split the data of each residue individually while maintaing the same label frequency in each split."
