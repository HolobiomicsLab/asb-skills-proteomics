---
name: word2vec-cbow-model-training
description: Use when you need to convert protein/GPCR primary sequences into fixed-length
  word-embedding features by splitting them into l-mers (l=2,3,4), training CBOW models
  on the resulting wordbooks, and aggregating word vectors per sequence.
when_to_use_negative:
- The input is already a numeric feature table (e.g., precomputed BOW or embedding
  features) — training CBOW models would be redundant; go straight to the downstream
  DL/XGBoost pipeline.
- The task is purely classifier selection (CatBoost vs XGBoost) rather than feature
  extraction — the embedding step produces only the original features that are fed
  onward.
- Sequences are not protein primary sequences (e.g., nucleotide or short peptides
  outside the GPCR fragment-length regime), so the l=2/3/4 wordbook assumptions do
  not apply.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3474
tools:
- name: Gensim
  role: Python package used to build and train the CBOW Word2vec models with Negative
    Sampling on each fragment-length wordbook
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/requirements.txt
- name: XGBoost
  role: Downstream Extreme Gradient Boosting classifier that consumes the features
    produced after the CBOW embedding and DL dimension-reduction step
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: GetCBOWFeatures.py
  role: Repository script containing the CBOW training code and the values of hyperparameters
    used in this skill
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:f4bf31e5cf6e07d7271e3e0b925467eb9bea6538c180aa71a6bdbab6724bd0e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - Gensim
  - GetCBOWFeatures.py
  - XGBoost
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# word2vec-cbow-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train Gensim Word2vec Continuous Bag-of-Words (CBOW) models with Negative Sampling on amino-acid fragment wordbooks of a GPCR sequence set, then use the trained models to embed each sequence into word-embedding features (EF2, EF3, EF4) that are concatenated into a fixed-length 896-D WEF vector.

## When to use

Use when you need to convert protein/GPCR primary sequences into fixed-length word-embedding features by splitting them into l-mers (l=2,3,4), training CBOW models on the resulting wordbooks, and aggregating word vectors per sequence.

## When NOT to use

- The input is already a numeric feature table (e.g., precomputed BOW or embedding features) — training CBOW models would be redundant; go straight to the downstream DL/XGBoost pipeline.
- The task is purely classifier selection (CatBoost vs XGBoost) rather than feature extraction — the embedding step produces only the original features that are fed onward.
- Sequences are not protein primary sequences (e.g., nucleotide or short peptides outside the GPCR fragment-length regime), so the l=2/3/4 wordbook assumptions do not apply.

## Inputs

- GPCR benchmark protein primary sequences (e.g., a FASTA file of GPCR sequences)
- Wordbooks of amino-acid fragments for each fragment length l=2, l=3, l=4
- Hyperparameter settings: hidden vector size N in {128, 256, 512}, iteration budget per wordbook

## Outputs

- Trained Gensim CBOW model checkpoints for each wordbook (Ql=2 at the 74th training, Ql=3 at the 100th, Ql=4 at the 280th)
- Per-sequence word-embedding fragment features EF2, EF3, EF4
- Concatenated fixed-length 896-D WEF feature matrix (wef_features.csv)

## How to apply

Load the GPCR benchmark sequences and split each sequence into fragments (words) of length l=2, 3, and 4 to build three wordbooks. Train a Word2vec CBOW model with Negative Sampling on each wordbook using Gensim, sweeping the hidden vector size N over {128, 256, 512} per fragment length. Set a large iteration budget and pick the default model per wordbook by monitoring convergence of the CBOW objective function: the increment becomes close to 0 after 74 iterations for l=2, 100 for l=3, and 280 for l=4, so those checkpoints were selected. Input each GPCR sequence into its trained CBOW model to obtain word vectors, aggregate them into per-sequence fragment features EF2, EF3, and EF4, and concatenate them into a single 896-D WEF vector (written to wef_features.csv). The trained models and code with hyperparameter values live in GetCBOWFeatures.py of the EMCBOW-GPCR repository; the first-time user should run GetResults.py in a python 3.7.0 virtual environment with packages from requirements.txt to generate models.

## Related tools

- **Gensim** (Python package used to build and train the CBOW Word2vec models with Negative Sampling on each fragment-length wordbook) — https://github.com/454170054/EMCBOW-GPCR/blob/main/requirements.txt
- **XGBoost** (Downstream Extreme Gradient Boosting classifier that consumes the features produced after the CBOW embedding and DL dimension-reduction step) — https://github.com/454170054/EMCBOW-GPCR
- **GetCBOWFeatures.py** (Repository script containing the CBOW training code and the values of hyperparameters used in this skill) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py

## Examples

```
python GetCBOWFeatures.py  # trains the CBOW models (see code/GetCBOWFeatures.py for hyperparameters); run inside a python 3.7.0 env with packages from requirements.txt, after GetResults.py
```

## Evaluation signals

- Convergence check: the CBOW objective function increment is close to 0 after the selected iteration checkpoints (74 for Ql=2, 100 for Ql=3, 280 for Ql=4); choosing earlier iterations would indicate undertraining.
- Dimensionality check: concatenated EF2 + EF3 + EF4 features must yield a single 896-D WEF vector per sequence.
- Hyperparameter sweep coverage: hidden vector size N must have been swept over {128, 256, 512} for each fragment length.
- Downstream benchmark: features from this step combined with BOW features should support CatBoost AUC = 0.9423 (vs 0.9391 for CBOW features alone and 0.9344 for BOW alone).
- Environment check: python 3.7.0 virtual environment with packages from requirements.txt installed, and GetResults.py run first to generate models before prediction.

## Limitations

- Version numbers for the software tools (e.g., Gensim, CatBoost, XGBoost) are not stated in the article; they must be recovered from the repository's requirements.txt.
- The hyperparameter N (hidden vector size in Word Embedding) is flexible and hard to determine the best value, and may significantly influence the quality of the features.
- The best iteration is chosen empirically by observing objective-function convergence rather than by a formal stopping criterion, so checkpoints (74/100/280) may not generalize to other datasets.

## Evidence
