---
name: roc-auc-evaluation
description: Use when comparing S-nitrosylation site predictors or deep-learning classifiers
  on the same test set and you need a threshold-independent goodness measure (AUC)
  alongside threshold-dependent Sn/Sp/Acc/MCC, e.g. on the independent test set or
  ten-fold CV predictions.
when_to_use_negative:
- Only hard thresholded labels are available (e.g. baseline predictors that report
  only Sn/Sp/Acc/MCC without score outputs), so no threshold-independent curve can
  be traced
- The question is calibration or absolute probability quality rather than ranking
  performance, where AUC is insensitive to class-imbalance-driven score distortion
- Predictors were trained/evaluated on non-redundancy-controlled datasets, making
  a same-test-set AUC comparison invalid (CD-HIT 40% clustering must precede test-set
  construction)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SNO-DCA
  role: Produces the softmax class probabilities per SNO-site peptide (one-hot 41
    x 21 input through 5 dense blocks, ECA-Net, FC 80/40/40/20) from which the ROC
    curve and AUC are computed
  repo: https://github.com/peanono/SNO-DCA
- name: CD-HIT
  role: Clusters sequences at a 40% threshold to build a redundancy-reduced independent
    test set so the computed AUC reflects generalization rather than sequence homology
    leakage
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:b2aad33e4959edf5f42dcfedafab1cfff8f9e2ad379baf92f53145956fda54d6
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
  - PreSNO
  - RecSNO
  - SNO-DCA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# roc-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Threshold-independent evaluation of SNO site predictors by computing ROC curves and the area under the curve (AUC) over predicted softmax probabilities, as used to rank SNO-DCA against DeepNitro, GPS-SNO, iSNO-PseAAC, PreSNO, RecSNO, and SNOSite/NASNO-Site.

## When to use

Use when comparing S-nitrosylation site predictors or deep-learning classifiers on the same test set and you need a threshold-independent goodness measure (AUC) alongside threshold-dependent Sn/Sp/Acc/MCC, e.g. on the independent test set or ten-fold CV predictions.

## When NOT to use

- Only hard thresholded labels are available (e.g. baseline predictors that report only Sn/Sp/Acc/MCC without score outputs), so no threshold-independent curve can be traced
- The question is calibration or absolute probability quality rather than ranking performance, where AUC is insensitive to class-imbalance-driven score distortion
- Predictors were trained/evaluated on non-redundancy-controlled datasets, making a same-test-set AUC comparison invalid (CD-HIT 40% clustering must precede test-set construction)

## Inputs

- Softmax predicted class probabilities for each peptide fragment from the SNO-DCA model
- Binary ground-truth labels for the independent test set (365 positive SNO sites / 3354 negative sites after CD-HIT 40% clustering)
- Cross-validation predictions from ten-fold CV on the benchmark dataset (3409 positive / 17103 negative training sites)
- Published baseline AUC values for comparison (RecSNO 0.80, PreSNO 0.76, DeepNitro 0.73, GPS-SNO 0.52)

## Outputs

- ROC curve over predicted probabilities
- AUC value per evaluation split (SNO-DCA: 0.77 ten-fold CV; 0.81-0.82 independent test)
- Predictor-comparison table (comparison_table.csv) with Sn, Sp, Acc, MCC, AUC columns

## How to apply

Run the predictor (e.g. SNO-DCA: 5 dense convolutional blocks + ECA-Net attention + four fully connected layers of 80/40/40/20 units) in inference mode so every peptide — one-hot encoded as a 41 x 21 matrix from the sliding window L = 2*xi+1 with xi = 20 and X-padding — yields a softmax class probability rather than a hard label. Compute the ROC curve from these probabilities and integrate to obtain AUC (Equations 12-15 territory covers the companion TP/FP-based metrics). Evaluate on the CD-HIT-40%-reduced independent test set (365 positive, 3354 negative sites from Xie et al.) and/or on ten-fold cross-validation predictions using the 5:1 positive:negative class weights during training. Compare the resulting AUC against published baselines (RecSNO AUC 0.80, PreSNO 0.76, DeepNitro 0.73, GPS-SNO 0.52) to judge whether the model attains the claimed superiority under imbalanced data.

## Related tools

- **SNO-DCA** (Produces the softmax class probabilities per SNO-site peptide (one-hot 41 x 21 input through 5 dense blocks, ECA-Net, FC 80/40/40/20) from which the ROC curve and AUC are computed) — https://github.com/peanono/SNO-DCA
- **CD-HIT** (Clusters sequences at a 40% threshold to build a redundancy-reduced independent test set so the computed AUC reflects generalization rather than sequence homology leakage)

## Examples

```
from sklearn.metrics import roc_curve, auc; fpr, tpr, _ = roc_curve(y_true, y_prob); auc(fpr, tpr)
```

## Evaluation signals

- Independent-test AUC should reproduce the reported value of ~0.81 (Table 4; text reports 0.82) within a tolerance of +/-0.02
- On the independent test set, computed AUC must exceed the strongest baseline RecSNO (AUC 0.80), supporting the claim of highest AUC among compared predictors
- Ablation ordering must hold under ten-fold CV: CNN baseline AUC 0.60 < CNN+DenseNets < full DenseNets+ECA-Net AUC 0.77
- AUC must be computed from continuous softmax probabilities, not hard 0/1 predictions; verify by checking the ROC curve is traced over multiple thresholds
- Metrics must be computed on the CD-HIT-40%-reduced test set with 365 positives and 3354 negatives; count mismatches invalidate the result

## Limitations

- Data imbalance (365 positives vs 3354 negatives) remains a major difficulty; the authors note 'the problem of data imbalance is still a major difficulty in the field of bioinformatics', so AUC alone can mask poor specificity at operationally relevant thresholds
- Table 4 (AUC 0.81) and the results text (AUC 0.82) differ slightly for the independent test set, so expect rounding-level ambiguity when validating
- Some baselines (iSNO-PseAAC, NASNO-Site) report NA for AUC, restricting the threshold-independent comparison to a subset of predictors

## Evidence
