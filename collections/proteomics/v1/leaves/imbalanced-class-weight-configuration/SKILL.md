---
name: imbalanced-class-weight-configuration
description: Use when training a peptide/site classifier on imbalanced positive:negative
  data (e.g., few SNO sites vs many non-SNO sites) and you need loss re-weighting
  before softmax output rather than resampling.
when_to_use_negative:
- Datasets are already balanced or have been rebalanced by resampling — the 5:1 weighting
  would distort the loss
- Multiclass problems with more than the two classes (SNO vs non-SNO) the 5:1 ratio
  was tuned for
- End-to-end prediction on raw sequences is required — the article notes the current
  model still requires pre-encoding of protein sequences
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SNO-DCA
  role: Deep predictor (DenseNet blocks + ECA-Net attention + FC layers + softmax)
    whose training loss is configured with the 5:1 class weights
  repo: https://github.com/peanono/SNO-DCA
- name: CD-HIT
  role: Preprocessing step that reduces sequence redundancy at a 40% threshold before
    class-imbalanced training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:47c30beb655f94d8842ad7483710eb0e0ca89a072abbc0b2483912b516165de4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# imbalanced-class-weight-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure training loss with class weights at a 5:1 positive:negative ratio so that a deep learning classifier (SNO-DCA) can learn S-nitrosylation sites despite severe imbalance between SNO and non-SNO samples.

## When to use

Use when training a peptide/site classifier on imbalanced positive:negative data (e.g., few SNO sites vs many non-SNO sites) and you need loss re-weighting before softmax output rather than resampling.

## When NOT to use

- Datasets are already balanced or have been rebalanced by resampling — the 5:1 weighting would distort the loss
- Multiclass problems with more than the two classes (SNO vs non-SNO) the 5:1 ratio was tuned for
- End-to-end prediction on raw sequences is required — the article notes the current model still requires pre-encoding of protein sequences

## Inputs

- One-hot encoded peptide windows (41 x 21 sparse matrices, window L = 41 with xi = 20, missing residues filled with 'X')
- Binary labels for SNO vs non-SNO classes with unequal positive:negative counts

## Outputs

- Trained two-class softmax classifier with loss re-weighted 5:1 positive:negative
- Cross-validation and independent-test performance metrics for the imbalanced SNO site prediction task

## How to apply

After building the network (dense convolutional blocks, ECA-Net attention, four fully connected layers of 80, 40, 40 and 20 units), attach a softmax output over two classes (SNO vs non-SNO) and pass a class-weight mapping to the training loss in which the minority positive class is weighted 5x the majority negative class. This was chosen because the SNO-DCA benchmark (4762 SNO sites from 3113 proteins, CD-HIT-clustered at 40%) is imbalanced, and the authors state class weights were introduced specifically to solve the problem of data imbalance. Verify empirically by 10-fold cross-validation and independent testing that metrics improve relative to unweighted training.

## Related tools

- **SNO-DCA** (Deep predictor (DenseNet blocks + ECA-Net attention + FC layers + softmax) whose training loss is configured with the 5:1 class weights) — https://github.com/peanono/SNO-DCA
- **CD-HIT** (Preprocessing step that reduces sequence redundancy at a 40% threshold before class-imbalanced training)

## Examples

```
model.fit(X_onehot, y, class_weight={0: 1.0, 1: 5.0}, epochs=50, batch_size=64)  # 5:1 positive:negative weighting per SNO-DCA
```

## Evaluation signals

- The loss function is instantiated with an explicit class-weight mapping reflecting the 5:1 positive:negative ratio
- The model accepts 41 x 21 one-hot input and emits a 2-class softmax output end-to-end without error
- Ten-fold cross-validation metrics (e.g., MCC, sensitivity, specificity) improve over the unweighted baseline, as reported when class weights were introduced
- Independent testing on held-out data shows the model outperforms existing imbalanced-data SNO predictors (DeepNitro, PreSNO, RecSNO, GPS-SNO, ISNOPseAAC, NASNO-Site)

## Limitations

- The 5:1 ratio is dataset-specific (4762 SNO sites from 3113 proteins); other imbalanced datasets may need ratio retuning
- Data imbalance remains an unresolved difficulty for the field even after class weighting
- Class weights address loss imbalance but the model still requires manual one-hot pre-encoding rather than end-to-end sequence input

## Evidence

- [methods] 5:1 class weight ratio in training loss: "we adopt the method of class weights and set the weight ratio of positive and negative samples to 5:1"
- [intro] class weights introduced to solve data imbalance before softmax: "the introduction of class weights to solve the problem of data imbalance, and finally softmax classification"
- [discussion] data imbalance remains an open field-level difficulty: "At present, the problem of data imbalance is still a major difficulty in the field of bioinformatics."
- [abstract] outperforms existing predictors under imbalanced data: "The 10-fold cross-validation and independent testing experimental results show that our SNO-DCA model outperforms existing S-nitrosylation sites prediction models under imbalanced data."
