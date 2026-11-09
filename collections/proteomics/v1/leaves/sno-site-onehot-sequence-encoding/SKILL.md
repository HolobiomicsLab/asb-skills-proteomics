---
name: sno-site-onehot-sequence-encoding
description: Use when preparing peptide windows around candidate SNO sites for a deep-learning
  classifier (e.g. SNO-DCA) and the input is raw protein sequences, not a precomputed
  feature table.
when_to_use_negative:
- The downstream model accepts embedded raw sequences (end-to-end prediction) — the
  paper notes the current model requires pre-encoding rather than end-to-end prediction
- The input is already a numeric feature table (e.g. handcrafted physicochemical features);
  re-encoding one-hot would discard existing features
- The task requires a different window length than ξ = 20; the 41 × 21 matrix shape
  is tied to this sliding-window setting
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_3474
tools:
- name: SNO-DCA
  role: Downstream model consuming the one-hot encoded 41 × 21 matrices as input for
    SNO site classification
  repo: https://github.com/peanono/SNO-DCA
- name: CD-HIT
  role: Pre-encoding redundancy reduction of collected protein sequences at a 40%
    clustering threshold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:8f26d8366de15c8c764fc35ab32fbf4d26bb968d8cd9a138a55ddc7fa3564783
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sno-site-onehot-sequence-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode S-nitrosylation site peptide fragments as 41 × 21 one-hot sparse matrices (861-dimensional flattened vectors) so that deep models like SNO-DCA can consume raw sequence windows without manual feature engineering, padding truncated windows with the X residue.

## When to use

Use when preparing peptide windows around candidate SNO sites for a deep-learning classifier (e.g. SNO-DCA) and the input is raw protein sequences, not a precomputed feature table.

## When NOT to use

- The downstream model accepts embedded raw sequences (end-to-end prediction) — the paper notes the current model requires pre-encoding rather than end-to-end prediction
- The input is already a numeric feature table (e.g. handcrafted physicochemical features); re-encoding one-hot would discard existing features
- The task requires a different window length than ξ = 20; the 41 × 21 matrix shape is tied to this sliding-window setting

## Inputs

- Protein sequences with annotated SNO sites (benchmark dataset from Xie et al. [14]: 4762 SNO sites from 3113 proteins)
- Independent test set peptides (365 positive sites and 3354 negative sites)
- Sliding window parameters: ξ = 20, window size L = 41

## Outputs

- One-hot encoded 41 × 21 (861-dimensional) sparse matrices, one per peptide fragment
- Encoded matrix arrays ready for dense convolutional network input

## How to apply

First extract fragments around each SNO site using a sliding window of size L = 2ξ+1 with ξ = 20 (L = 41); if the protein terminus truncates the window, fill missing amino acids with X. Then encode each L-length fragment as a two-dimensional sparse matrix of L × 21 dimensions (one-hot per position over 20 amino acids plus padding), yielding 41 × 21 = 861 features per peptide. Rationale: the paper explicitly reduces manual feature-extraction intervention by relying solely on one-hot encoding, letting the downstream dense convolutional blocks and ECA-Net attention learn high-level features. Before encoding, sequences should be redundancy-reduced with CD-HIT at a 40% threshold to lower overfitting risk. For imbalanced training sets (positives far fewer than negatives), apply class weights (5:1 positive:negative) at the classifier stage, not during encoding.

## Related tools

- **SNO-DCA** (Downstream model consuming the one-hot encoded 41 × 21 matrices as input for SNO site classification) — https://github.com/peanono/SNO-DCA
- **CD-HIT** (Pre-encoding redundancy reduction of collected protein sequences at a 40% clustering threshold)

## Examples

```
python encode_peptides.py --fasta proteins.fasta --sites sno_sites.csv --xi 20 --pad X --output encoded_41x21.npz
```

## Evaluation signals

- Every encoded peptide matrix has shape 41 × 21 (861 features when flattened); any other shape indicates wrong ξ or padding
- Fragments shorter than 41 residues contain X characters in truncated positions, and X is encoded as its own one-hot column
- Encoded dataset sizes match expectations: independent test set = 365 positive + 3354 negative peptides
- Downstream SNO-DCA metrics on the encoded test set reproduce Sn ≈ 0.82, Sp ≈ 0.69, Acc ≈ 0.70, MCC ≈ 0.32, AUC ≈ 0.81 within ±0.02
- Sequences were clustered with CD-HIT at 40% identity before encoding, so no pair of encoded training peptides derives from sequences above that redundancy

## Limitations

- Current model requires pre-encoding of protein sequences rather than end-to-end prediction from embedded original sequences
- Data imbalance remains a major difficulty; encoding alone does not solve it — class weights (5:1) are needed downstream
- No software version numbers stated for the preprocessing tooling (e.g. CD-HIT version not given)

## Evidence
