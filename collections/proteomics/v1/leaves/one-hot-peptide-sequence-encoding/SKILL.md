---
name: one-hot-peptide-sequence-encoding
description: Use when raw protein sequences with candidate S-nitrosylation site positions
  must be converted to fixed-size numeric matrices to feed a CNN-based site predictor
  like SNO-DCA, and manual feature extraction should be avoided.
when_to_use_negative:
- When a richer encoding is already available (e.g. precomputed feature tables or
  embeddings) — the article's design deliberately uses ONLY one-hot encoding, so adding
  manual features contradicts its 'reduced manual feature-extraction intervention'
  rationale
- When a variable-length or end-to-end prediction model is required — the article
  notes the current model still requires sequences to be pre-encoded rather than embedded
  end-to-end
- When fragments shorter than the window cannot be padded acceptably — 'X'-padded
  positions occupy the 21st channel and carry no amino-acid information
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_3474
tools:
- name: SNO-DCA
  role: Downstream predictor whose first-stage input pipeline consumes the 41 × 21
    one-hot encoded fragments produced by this skill
  repo: https://github.com/peanono/SNO-DCA
- name: CD-HIT
  role: Pre-encoding redundancy reduction of collected protein sequences at a 40%
    clustering threshold before fragment extraction and one-hot encoding
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.heliyon.2023.e23187
    title: 'SNO-DCA: A model for predicting S-nitrosylation sites based on densely
      connected convolutional networks and attention mechanism'
  source_content_hash: sha256:e42428fc81e5ea849223dc79ae6a64d0bc3e31cc01dd9f1c9bdbcb20f99bd93b
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
n_claims: 9
---

# one-hot-peptide-sequence-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes fixed-length SNO-site peptide fragments (sliding window L = 2ξ+1 with ξ = 20) into 41 × 21 two-dimensional sparse one-hot matrices, padding missing residues with 'X', as the sole input representation for the SNO-DCA densely connected convolutional network.

## When to use

Use when raw protein sequences with candidate S-nitrosylation site positions must be converted to fixed-size numeric matrices to feed a CNN-based site predictor like SNO-DCA, and manual feature extraction should be avoided.

## When NOT to use

- When a richer encoding is already available (e.g. precomputed feature tables or embeddings) — the article's design deliberately uses ONLY one-hot encoding, so adding manual features contradicts its 'reduced manual feature-extraction intervention' rationale
- When a variable-length or end-to-end prediction model is required — the article notes the current model still requires sequences to be pre-encoded rather than embedded end-to-end
- When fragments shorter than the window cannot be padded acceptably — 'X'-padded positions occupy the 21st channel and carry no amino-acid information

## Inputs

- Protein sequences in standard amino-acid format with annotated candidate S-nitrosylation (SNO) sites
- Sliding-window parameters ξ = 20 (window size L = 41)
- Padding character 'X' for terminal-truncated fragments

## Outputs

- 41 × 21 one-hot sparse matrices, one per candidate site (861-dimensional input vectors)
- Encoded fragment set ready for SNO-DCA training (3409 positive and 17,103 negative fragments in the benchmark)

## How to apply

First reduce sequence redundancy with CD-HIT at a 40% threshold, then apply a sliding window of size L = 2ξ+1 with ξ = 20 centered on each candidate site to extract 41-residue fragments. Fill missing amino acids with 'X' when fragments extend past sequence termini, so every fragment has length L = 41. One-hot encode each fragment into an L × 21 dimensional two-dimensional sparse matrix (the 21st channel accounts for the 'X' padding), giving an 861-dimensional input vector per fragment. Rationale: the article explicitly reduced manual feature-extraction intervention by using only one-hot encoding, letting the downstream 5 dense convolutional blocks (kernel size 4, 96 filters, growth D″ = 32) learn high-level features, with ECA-Net attention weighting them before the 80/40/40/20 fully connected layers and softmax classifier with 5:1 class weights.

## Related tools

- **SNO-DCA** (Downstream predictor whose first-stage input pipeline consumes the 41 × 21 one-hot encoded fragments produced by this skill) — https://github.com/peanono/SNO-DCA
- **CD-HIT** (Pre-encoding redundancy reduction of collected protein sequences at a 40% clustering threshold before fragment extraction and one-hot encoding)

## Examples

```
python encode_fragments.py --sequences proteins.fasta --sites sno_sites.tsv --xi 20 --pad X --output fragments_41x21.npz
```

## Evaluation signals

- Every encoded fragment is exactly a 41 × 21 sparse matrix (L = 2ξ+1 with ξ = 20), i.e. an 861-dimensional flattened vector
- Fragments near sequence termini are length-normalized by filling missing amino acids with 'X', which maps to the 21st one-hot channel, not a zero vector
- The encoder output feeds the SNO-DCA model end-to-end: 41 × 21 input → initial conv layer (kernel size S = 4, D = 96 filters, ELU) without shape errors
- Downstream ten-fold cross-validation on the encoded benchmark (3409 positives, 17,103 negatives) reproduces Sn = 0.80, Sp = 0.62, Acc = 0.65, MCC = 0.31, AUC = 0.77 within reasonable tolerance
- Positive:negative fragment ratio (~1:5) matches the 5:1 class weighting used during training, confirming balanced handling of the imbalanced encoded dataset

## Limitations

- Only one-hot encoding is used, so no evolutionary or physicochemical information is captured; performance relies entirely on the dense convolutional blocks learning features
- The current model requires sequences to be pre-encoded in advance rather than achieving end-to-end prediction from raw sequences (a limitation the authors plan to address)
- 'X' padding of terminal fragments introduces non-informative channels that the model must learn to ignore
- Data imbalance (3409 positives vs 17,103 negatives) is only partly mitigated — handled downstream by 5:1 class weights, not by the encoding itself

## Evidence
