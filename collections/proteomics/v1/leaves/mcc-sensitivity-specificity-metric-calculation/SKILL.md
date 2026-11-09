---
name: mcc-sensitivity-specificity-metric-calculation
description: Use when comparing S-nitrosylation site predictors on a shared independent
  test set and you need Sn, Sp, Acc, MCC and AUC to assemble a comparison table under
  imbalanced positive:negative data.
when_to_use_negative:
- When the input is already an assembled comparison table of published metrics rather
  than raw model predictions or confusion-matrix counts
- When the evaluation set is balanced or differs from the CD-HIT-40%-clustered independent
  test set — the paper's baseline metric values are only comparable on the same 365/3354
  positive:negative split
- When predictors were trained on different benchmark datasets; cross-dataset MCC/AUC
  values are not directly comparable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0623
tools:
- name: SNO-DCA
  role: Predictor whose independent-test predictions yield Sn = 0.82, Sp = 0.69, Acc
    = 0.70, MCC = 0.32, AUC = 0.82
  repo: https://github.com/peanono/SNO-DCA
- name: RecSNO
  role: Strongest baseline predictor; published independent-test MCC 0.30, AUC 0.80
    used as the benchmark to beat
- name: PreSNO
  role: Baseline predictor (SVM and RF based); published independent-test MCC 0.25,
    AUC 0.76
- name: DeepNitro
  role: Baseline predictor (eight-layer neural network); published independent-test
    MCC 0.22, AUC 0.73
- name: GPS-SNO
  role: Baseline predictor; published independent-test MCC 0.01, AUC 0.52
- name: ISNOPseAAC
  role: Baseline predictor; published independent-test MCC 0.03, AUC not available
    (NA)
- name: NASNO-Site
  role: Baseline predictor; published independent-test MCC 0.07, AUC not available
    (NA)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:64e335d9a4295cdf17e0d4f500f3539adc2aedca6945b93c2a3230d2fd25aff9
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
  - DeepNitro
  - GPS-SNO
  - ISNOPseAAC
  - NASNO-Site
  - PreSNO
  - RecSNO
  - SNO-DCA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mcc-sensitivity-specificity-metric-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Sn, Sp, Acc, MCC and AUC to benchmark SNO-DCA against DeepNitro, GPS-SNO, iSNO-PseAAC, PreSNO, RecSNO and NASNO-Site on the independent test set (365 positive / 3354 negative SNO sites), enabling the claim that SNO-DCA attains the highest Sn, MCC and AUC under imbalanced data.

## When to use

Use when comparing S-nitrosylation site predictors on a shared independent test set and you need Sn, Sp, Acc, MCC and AUC to assemble a comparison table under imbalanced positive:negative data.

## When NOT to use

- When the input is already an assembled comparison table of published metrics rather than raw model predictions or confusion-matrix counts
- When the evaluation set is balanced or differs from the CD-HIT-40%-clustered independent test set — the paper's baseline metric values are only comparable on the same 365/3354 positive:negative split
- When predictors were trained on different benchmark datasets; cross-dataset MCC/AUC values are not directly comparable

## Inputs

- Independent test set derived from Xie et al. benchmark dataset (365 positive SNO sites, 3354 negative sites, CD-HIT clustered at 40% threshold)
- Model predictions (confusion-matrix counts) from SNO-DCA on the independent test set
- Published independent-test-set metric values for baseline predictors (DeepNitro, GPS-SNO, iSNO-PseAAC, PreSNO, RecSNO, NASNO-Site)

## Outputs

- Per-predictor table of Sn, Sp, Acc, MCC and AUC (comparison_table.csv)
- Validated ranking showing SNO-DCA attains the highest Sn (0.82), MCC (0.32) and AUC (0.82) among compared predictors

## How to apply

Run the trained SNO-DCA model (5 dense convolutional blocks, ECA-Net attention, four fully connected layers with 80/40/40/20 units and softmax) on the independent test set derived from Xie et al. — 365 positive SNO sites and 3354 negative sites retained after CD-HIT clustering at a 40% threshold — and compute sensitivity (Sn), specificity (Sp), accuracy (Acc), MCC and AUC from the confusion matrix and ROC curve. Collect the published independent-test metrics of the baselines: DeepNitro (MCC 0.22, AUC 0.73), GPS-SNO (MCC 0.01, AUC 0.52), iSNO-PseAAC (MCC 0.03), PreSNO (MCC 0.25, AUC 0.76), RecSNO (MCC 0.30, AUC 0.80) and NASNO-Site (MCC 0.07). SNO-DCA's expected values are Sn = 0.82, Sp = 0.69, Acc = 0.70, MCC = 0.32 and AUC = 0.82. Assemble all predictors' five metrics into a single table (e.g. comparison_table.csv) sorted by predictor. MCC is the key metric here because the positive:negative ratio is heavily imbalanced, so accuracy alone is misleading; the paper also handles this during training with 5:1 positive:negative class weights.

## Related tools

- **SNO-DCA** (Predictor whose independent-test predictions yield Sn = 0.82, Sp = 0.69, Acc = 0.70, MCC = 0.32, AUC = 0.82) — https://github.com/peanono/SNO-DCA
- **RecSNO** (Strongest baseline predictor; published independent-test MCC 0.30, AUC 0.80 used as the benchmark to beat)
- **PreSNO** (Baseline predictor (SVM and RF based); published independent-test MCC 0.25, AUC 0.76)
- **DeepNitro** (Baseline predictor (eight-layer neural network); published independent-test MCC 0.22, AUC 0.73)
- **GPS-SNO** (Baseline predictor; published independent-test MCC 0.01, AUC 0.52)
- **ISNOPseAAC** (Baseline predictor; published independent-test MCC 0.03, AUC not available (NA))
- **NASNO-Site** (Baseline predictor; published independent-test MCC 0.07, AUC not available (NA))

## Examples

```
import pandas as pd; from sklearn.metrics import matthews_corrcoef, roc_auc_score, recall_score, precision_score; pd.DataFrame(rows, columns=['predictor','Sn','Sp','Acc','MCC','AUC']).to_csv('comparison_table.csv', index=False)
```

## Evaluation signals

- SNO-DCA on the independent test set must yield Sn = 0.82, Sp = 0.69, Acc = 0.70, MCC = 0.32, AUC = 0.82 (Table 4 values)
- SNO-DCA must attain the highest Sn, MCC and AUC of all compared predictors (strongest baseline: RecSNO, MCC 0.30, AUC 0.80)
- Baseline rows must match published values: DeepNitro MCC 0.22/AUC 0.73, GPS-SNO MCC 0.01/AUC 0.52, iSNO-PseAAC MCC 0.03, PreSNO MCC 0.25/AUC 0.76, RecSNO MCC 0.30/AUC 0.80, NASNO-Site MCC 0.07
- Test-set composition must be 365 positive SNO sites and 3354 negative sites after CD-HIT clustering at a 40% threshold
- Rows with NA AUC (iSNO-PseAAC, NASNO-Site) must be handled explicitly, not filled with zeros

## Limitations

- Data imbalance remains a major difficulty in bioinformatics; accuracy is inflated on imbalanced sets, so MCC and AUC should carry the comparison weight
- Some baselines (iSNO-PseAAC, NASNO-Site) report no AUC (NA), limiting full-metric comparison
- Metric comparability depends on all predictors being evaluated on the same independent test set from Xie et al. benchmark (4762 SNO sites from 3113 proteins)
- Class weights (5:1 positive:negative) affect trained-model operating points, so metrics reflect that imbalance-handling choice

## Evidence

- [abstract] Independent test metrics claim: "The 10-fold cross-validation and independent testing experimental results show that our SNO-DCA model outperforms existing S-nitrosylation sites prediction models under imbalanced data."
- [methods] CD-HIT threshold for test-set construction: "the collected protein sequences were clustered using a CD-HIT with a threshold of 40 %"
- [methods] Class weighting rationale for imbalanced evaluation: "we adopt the method of class weights and set the weight ratio of positive and negative samples to 5:1"
- [results] Baseline published metrics in results table: "GPS-SNO0.280.740.690.010.52"
- [full_text] Independent test set composition from source card workflow: [evidence span withheld — anchor status: not_found]
