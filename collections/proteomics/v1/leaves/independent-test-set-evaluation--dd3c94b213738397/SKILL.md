---
name: independent-test-set-evaluation
description: Use when a trained S-nitrosylation site predictor must be scored on a
  held-out independent test set (Xie et al. benchmark, CD-HIT 40%) with Sn/Sp/Acc/MCC/AUC
  and benchmarked against baseline predictors.
when_to_use_negative:
- Estimating model performance during development — use ten-fold cross-validation
  on the training set (3409 positive / 17103 negative) instead; the independent test
  set must remain untouched for the final comparison.
- The candidate sequences have not been redundancy-reduced (CD-HIT at 40% threshold)
  — homologous overlap with training data would inflate the reported metrics.
- The question concerns only a change in input representation (e.g., trainable embedding
  vs one-hot encoding) rather than the baseline evaluation itself — that comparison
  reuses this protocol but additionally requires side-by-side tabulation of both variants
  under identical conditions.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SNO-DCA
  role: The trained predictor being evaluated on the independent test set to produce
    Sn/Sp/Acc/MCC/AUC metrics
  repo: https://github.com/peanono/SNO-DCA
- name: CD-HIT
  role: Clustering at 40% threshold to remove sequence redundancy before constructing
    the independent test set
- name: RecSNO
  role: Strongest baseline predictor for comparison (MCC 0.30, AUC 0.80 on the independent
    test set)
- name: DeepNitro
  role: Baseline predictor whose published independent-test-set metrics (MCC 0.22,
    AUC 0.73) populate the comparison table
- name: PreSNO
  role: Baseline predictor whose published independent-test-set metrics (MCC 0.25,
    AUC 0.76) populate the comparison table
- name: GPS-SNO
  role: Baseline predictor whose published independent-test-set metrics (MCC 0.01,
    AUC 0.52) populate the comparison table
- name: ISNOPseAAC
  role: Baseline predictor whose published independent-test-set metrics (MCC 0.03,
    AUC NA) populate the comparison table
- name: NASNO-Site
  role: Baseline predictor whose published independent-test-set metrics (MCC 0.07,
    AUC NA) populate the comparison table
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:7a153f9c83aac4644e258847679558947a2a978e0af142591b4919e21fce5e2b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/peanono/SNO-DCA
  repo_url: https://github.com/peanono/SNO-DCA
  tools:
  - CD-HIT
  - DeepNitro
  - GPS-SNO
  - ISNOPseAAC
  - NASNO-Site
  - PreSNO
  - RecSNO
  - SNO-DCA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# independent-test-set-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a trained SNO-DCA S-nitrosylation site predictor on a held-out independent test set (365 positive / 3354 negative sites, CD-HIT-reduced at 40%) and computes Sn, Sp, Acc, MCC, and AUC for side-by-side comparison against published predictors (DeepNitro, GPS-SNO, iSNO-PseAAC, PreSNO, RecSNO, NASNO-Site).

## When to use

Use when a trained S-nitrosylation site predictor must be scored on a held-out independent test set (Xie et al. benchmark, CD-HIT 40%) with Sn/Sp/Acc/MCC/AUC and benchmarked against baseline predictors.

## When NOT to use

- Estimating model performance during development — use ten-fold cross-validation on the training set (3409 positive / 17103 negative) instead; the independent test set must remain untouched for the final comparison.
- The candidate sequences have not been redundancy-reduced (CD-HIT at 40% threshold) — homologous overlap with training data would inflate the reported metrics.
- The question concerns only a change in input representation (e.g., trainable embedding vs one-hot encoding) rather than the baseline evaluation itself — that comparison reuses this protocol but additionally requires side-by-side tabulation of both variants under identical conditions.

## Inputs

- Independent test set: 365 positive SNO site fragments and 3354 negative fragments, derived from the Xie et al. benchmark after CD-HIT clustering at 40% threshold
- Trained SNO-DCA model (5 dense convolutional blocks, ECA-Net attention, FC layers 80/40/40/20, softmax, class weights 5:1)
- Published independent-test-set metrics of baseline predictors (DeepNitro, GPS-SNO, iSNO-PseAAC, PreSNO, RecSNO, NASNO-Site)

## Outputs

- comparison_table.csv with Sn, Sp, Acc, MCC, and AUC for each predictor on the independent test set
- SNO-DCA reference metrics row (expected Sn = 0.82, Sp = 0.69, Acc = 0.70, MCC = 0.32, AUC = 0.82)

## How to apply

Load the independent test set derived from the Xie et al. benchmark (4762 SNO sites from 3113 proteins) after CD-HIT clustering at a 40% threshold, retaining 365 positive and 3354 negative sites. Extract 41-residue fragments (L = 2ξ+1, ξ = 20, missing residues padded with X), one-hot encode to 41 × 21 matrices, and run the trained SNO-DCA model (5 dense blocks, ECA-Net attention, FC layers 80/40/40/20 with softmax) to obtain predictions. Compute Sn, Sp, Acc, MCC, and AUC from the confusion counts, then assemble these metrics alongside the published independent-test-set values of the baselines (DeepNitro MCC 0.22/AUC 0.73; GPS-SNO MCC 0.01/AUC 0.52; iSNO-PseAAC MCC 0.03; PreSNO MCC 0.25/AUC 0.76; RecSNO MCC 0.30/AUC 0.80; NASNO-Site MCC 0.07) into a single table sorted by predictor and saved as comparison_table.csv. Accept the evaluation when SNO-DCA attains the highest Sn, MCC, and AUC among all compared predictors; the same protocol serves as the reference for variant comparisons (e.g., embedding vs one-hot input) that differ solely in input representation.

## Related tools

- **SNO-DCA** (The trained predictor being evaluated on the independent test set to produce Sn/Sp/Acc/MCC/AUC metrics) — https://github.com/peanono/SNO-DCA
- **CD-HIT** (Clustering at 40% threshold to remove sequence redundancy before constructing the independent test set)
- **RecSNO** (Strongest baseline predictor for comparison (MCC 0.30, AUC 0.80 on the independent test set))
- **DeepNitro** (Baseline predictor whose published independent-test-set metrics (MCC 0.22, AUC 0.73) populate the comparison table)
- **PreSNO** (Baseline predictor whose published independent-test-set metrics (MCC 0.25, AUC 0.76) populate the comparison table)
- **GPS-SNO** (Baseline predictor whose published independent-test-set metrics (MCC 0.01, AUC 0.52) populate the comparison table)
- **ISNOPseAAC** (Baseline predictor whose published independent-test-set metrics (MCC 0.03, AUC NA) populate the comparison table)
- **NASNO-Site** (Baseline predictor whose published independent-test-set metrics (MCC 0.07, AUC NA) populate the comparison table)

## Examples

```
from sklearn.metrics import matthews_corrcoef, roc_auc_score; sn = tp/(tp+fn); sp = tn/(tn+fp); acc = (tp+tn)/(tp+fn+tn+fp); mcc = matthews_corrcoef(y_true, y_pred); auc = roc_auc_score(y_true, y_score)
```

## Evaluation signals

- SNO-DCA metrics on the independent test set match the reference values Sn = 0.82, Sp = 0.69, Acc = 0.70, MCC = 0.32, AUC = 0.82.
- Test set composition is 365 positive and 3354 negative sites after CD-HIT clustering at the 40% threshold.
- SNO-DCA attains the highest Sn, MCC, and AUC among all compared predictors, exceeding the strongest baseline RecSNO (MCC 0.30, AUC 0.80).
- Baseline metrics in comparison_table.csv match published values (e.g., DeepNitro MCC 0.22/AUC 0.73, PreSNO MCC 0.25/AUC 0.76, GPS-SNO MCC 0.01/AUC 0.52, NASNO-Site MCC 0.07).
- Metrics are computed under the same 41-residue window (ξ = 20, X-padding) protocol used for training, so only the dataset differs from the cross-validation setting.

## Limitations

- Data imbalance remains a major difficulty in the field, and the class-weight scheme (5:1) may bias Sn vs Sp trade-offs on the test set.
- Data are only available on request, not publicly deposited, limiting independent reproduction of the test-set evaluation.
- Some baseline predictors report NA for AUC (iSNO-PseAAC, NASNO-Site), preventing complete metric-by-metric comparison.
- No software version numbers (e.g., for CD-HIT) are stated, which may affect exact reproducibility of the redundancy reduction.

## Evidence

- [abstract] The 10-fold cross-validation and independent testing experimental results show that our SNO-DCA model outperforms existing S-nitrosylation sites prediction models under imbalanced data.: "The 10-fold cross-validation and independent testing experimental results show that our SNO-DCA model outperforms existing S-nitrosylation sites prediction models under imbalanced data."
- [methods] the collected protein sequences were clustered using a CD-HIT with a threshold of 40 %: "the collected protein sequences were clustered using a CD-HIT with a threshold of 40 %"
- [methods] We used a sliding window to extract the fragments with a window size L = 2ξ+1. In this paper, we set ξ = 20 and fill the missing amino acids with X: "We used a sliding window to extract the fragments with a window size L = 2ξ+1. In this paper, we set ξ = 20 and fill the missing amino acids with X"
- [discussion] At present, the problem of data imbalance is still a major difficulty in the field of bioinformatics.: "At present, the problem of data imbalance is still a major difficulty in the field of bioinformatics."
