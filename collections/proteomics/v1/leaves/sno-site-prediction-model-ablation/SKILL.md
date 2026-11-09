---
name: sno-site-prediction-model-ablation
description: Use when you need to demonstrate that individual deep-learning components
  (dense convolutional blocks, channel attention) measurably improve site-prediction
  performance over a plain CNN baseline, using ten-fold cross-validation on sliding-window
  peptide fragments.
when_to_use_negative:
- The goal is benchmarking against external published predictors (DeepNitro, PreSNO,
  GPS-SNO, etc.) on the independent test set rather than internal component-wise ablation.
- Only the full SNO-DCA model is available and baseline variants (plain CNN, CNN without
  ECA-Net) cannot be constructed, so no component contribution can be isolated.
- Input sequences have not been redundancy-reduced with CD-HIT at the 40% threshold,
  risking inflated cross-validation scores from homologous sequences leaking across
  folds.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: SNO-DCA
  role: 'Full model under ablation: DenseNets feature extraction plus ECA-Net attention,
    evaluated against reduced variants'
  repo: https://github.com/peanono/SNO-DCA
- name: CD-HIT
  role: Pre-ablation sequence redundancy reduction at 40% clustering threshold to
    prevent overfitting from homologous proteins
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:8799f094bd3d71172826575c7faa5919c11f2ee810912183e140c1ed483192cd
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
  - SNO-DCA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# sno-site-prediction-model-ablation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ablation protocol for S-nitrosylation site predictors that compares a conventional CNN baseline against a CNN+DenseNets variant and the full DenseNets+ECA-Net (SNO-DCA) architecture under ten-fold cross-validation, quantifying each component's contribution via Sn, Sp, Acc, MCC and AUC.

## When to use

Use when you need to demonstrate that individual deep-learning components (dense convolutional blocks, channel attention) measurably improve site-prediction performance over a plain CNN baseline, using ten-fold cross-validation on sliding-window peptide fragments.

## When NOT to use

- The goal is benchmarking against external published predictors (DeepNitro, PreSNO, GPS-SNO, etc.) on the independent test set rather than internal component-wise ablation.
- Only the full SNO-DCA model is available and baseline variants (plain CNN, CNN without ECA-Net) cannot be constructed, so no component contribution can be isolated.
- Input sequences have not been redundancy-reduced with CD-HIT at the 40% threshold, risking inflated cross-validation scores from homologous sequences leaking across folds.

## Inputs

- CD-HIT-reduced (40% threshold) S-nitrosylation benchmark dataset from Xie et al. [14] (3409 positive / 17,103 negative training sites)
- Sliding-window peptide fragments of length L = 41 (ξ = 20, missing residues padded with X)
- One-hot encoded 41 × 21 sparse matrices per fragment
- Three model configurations: CNN baseline, CNN+DenseNets, full DenseNets+ECA-Net (SNO-DCA)

## Outputs

- ablation_metrics.csv — table of Sn, Sp, Acc, MCC and ROC-AUC per model variant under ten-fold cross-validation
- Per-fold cross-validation predictions for each variant

## How to apply

Load the CD-HIT-reduced (40% threshold) benchmark from Xie et al. (3409 positive / 17,103 negative sites) and one-hot encode the L = 41 sliding-window fragments (ξ = 20, missing residues padded with X) into 41 × 21 sparse matrices. Build three variants: (1) a CNN baseline with a 1D convolution of 96 filters and kernel size S = 4 followed by four fully connected layers (80, 40, 40, 20 units) and softmax; (2) the same model plus 5 densely connected blocks (32 kernels per dense layer, 4 convolution layers per block, transition layers with 1D average pooling); (3) the full SNO-DCA model additionally appending the ECA-Net channel attention module (GAP compression, adaptive 1D convolution kernel size with γ = 2, b = 1). Train each variant with ten-fold cross-validation using class weights of 5:1 (positive:negative) to counter the ~1:5 data imbalance, then compute Sn, Sp, Acc, MCC and ROC-AUC from the pooled cross-validated predictions. The ablation is correct only if all five metrics improve monotonically from baseline → CNN+DenseNets → full model; the paper's reference result is Sn 0.80, Sp 0.62, Acc 0.65, MCC 0.31, AUC 0.77 for the full model versus Sn 0.32, MCC 0.09, AUC 0.60 for the CNN baseline. Save the per-variant comparison to ablation_metrics.csv.

## Related tools

- **SNO-DCA** (Full model under ablation: DenseNets feature extraction plus ECA-Net attention, evaluated against reduced variants) — https://github.com/peanono/SNO-DCA
- **CD-HIT** (Pre-ablation sequence redundancy reduction at 40% clustering threshold to prevent overfitting from homologous proteins)

## Evaluation signals

- All five metrics (Sn, Sp, Acc, MCC, AUC) improve monotonically across the three variants: CNN baseline → CNN+DenseNets → full DenseNets+ECA-Net.
- Full-model ten-fold CV metrics fall near the reference values Sn 0.80, Sp 0.62, Acc 0.65, MCC 0.31, AUC 0.77.
- CNN baseline metrics fall near the reported Sn 0.32, MCC 0.09, AUC 0.60, confirming the baseline is not accidentally as strong as the full model.
- Each variant was trained with the identical protocol: ten folds and class weights of 5:1 positive:negative — any metric gap must be attributable to architecture, not training conditions.
- ablation_metrics.csv contains exactly three variant rows with all five metric columns populated (AUC 'NA' permitted for weaker baselines is acceptable per comparator conventions).

## Limitations

- Data imbalance remains an unresolved difficulty: class weighting (5:1) mitigates but does not solve the ~1:5 positive:negative ratio, and specificity for the full model (0.62) still lags sensitivity (0.80).
- Hyperparameters (5 dense blocks, 4 layers per block, 96 first-layer filters) were fixed by grid exploration in the paper; re-running the ablation with different settings may shift the component contributions.
- CD-HIT version and software version numbers are not stated in the article, complicating exact reproduction.
- The ablation uses cross-validation only; independent test-set comparison against other predictors is a separate exercise.

## Evidence
