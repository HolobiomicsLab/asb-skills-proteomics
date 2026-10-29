---
name: cbow-word-embedding-feature-generation
description: Use when raw GPCR amino-acid sequences must be converted into dense numerical
  feature vectors (word embeddings) for downstream classification, e.g. feeding CatBoost/XGBoost
  in a GPCR identification task.
when_to_use_negative:
- Input is already a numeric feature table (embedding generation would be redundant);
  proceed directly to classifier training
- Sequences are not protein/peptide text — the CBOW models are trained on split GPCR
  sequence segments and the learned vocabulary will not transfer
- The goal is BOW-only features rather than word-embedding features
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0080
tools:
- name: Gensim
  role: Python package used to build and train the CBOW word-embedding models on split
    GPCR sequence segments
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:5ebfec693c6839ab964275686fdc67eca9e30aa5200f7908760c8c3758ac5a89
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cbow-word-embedding-feature-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate continuous bag-of-words (CBOW) word-embedding features from GPCR protein sequences by splitting sequences into length-labeled segments (Ql=2/3/4) and training Gensim CBOW models, as done in the EMCBOW-GPCR pipeline.

## When to use

Use when raw GPCR amino-acid sequences must be converted into dense numerical feature vectors (word embeddings) for downstream classification, e.g. feeding CatBoost/XGBoost in a GPCR identification task.

## When NOT to use

- Input is already a numeric feature table (embedding generation would be redundant); proceed directly to classifier training
- Sequences are not protein/peptide text — the CBOW models are trained on split GPCR sequence segments and the learned vocabulary will not transfer
- The goal is BOW-only features rather than word-embedding features

## Inputs

- GPCR protein sequences in FASTA format (e.g. test.fasta)
- Benchmark GPCR dataset with label file (from the EMCBOW-GPCR repository)

## Outputs

- CBOW word-embedding feature vectors, one per input sequence
- Trained Gensim CBOW models per segment length (Ql=2, Ql=3, Ql=4)

## How to apply

First split each GPCR sequence into segments of fixed length (the article uses segment length classes Ql=2, Ql=3, Ql=4) and train a CBOW word-embedding model on the split segments using the Gensim Python package with the hyperparameters encoded in the repository's GetCBOWFeatures.py. Train with a large iteration budget and select the best model by observing convergence of the CBOW objective function: for Ql=2 the increment approaches 0 after 74 iterations, so the 74th training is the default model (100th and 280th for Ql=3 and Ql=4 per the article). Then input each GPCR sequence into the trained embedding model to obtain its word vectors. These CBOW features can be used alone or concatenated with BOW features (combined representation reached AUC 0.9423 with CatBoost). Run the code in a Python 3.7.0 virtual environment with the packages pinned in requirements.txt.

## Related tools

- **Gensim** (Python package used to build and train the CBOW word-embedding models on split GPCR sequence segments) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py

## Examples

```
python code/GetCBOWFeatures.py  # within the Python 3.7.0 env built from https://github.com/454170054/EMCBOW-GPCR
```

## Evaluation signals

- The CBOW objective function increment is close to 0 at the selected iteration (74th training for Ql=2; 100th and 280th for Ql=3 and Ql=4)
- Downstream CatBoost 5-fold cross-validation (20 repeats) on CBOW features reproduces AUC ≈ 0.9391 on the benchmark dataset
- Environment matches README requirements: Python 3.7.0 virtual environment with packages from requirements.txt installed
- Feature vectors have one entry per input sequence and the count matches the number of sequences in the benchmark dataset
- Concatenating generated CBOW vectors with BOW vectors yields combined features whose CatBoost AUC (0.9423) exceeds BOW-only (0.9344) and CBOW-only (0.9391)

## Limitations

- The word-embedding hyperparameter N is flexible and hard to determine; it may significantly influence feature quality
- Version numbers for Gensim and other tools are not stated in the paper; only requirements.txt in the repository pins them
- The best iteration per segment length (Ql) must be selected empirically by observing objective-function convergence, adding a manual decision point

## Evidence

- [intro] Split GPCR sequences into segments of different lengths and train corresponding word embedding models with the split segments.: "Firstly, we split the GPCRs sequences into segments of different lengths, and train the corresponding word embedding model with the split segments."
- [results] Choose the 74th training as the default CBOW model of Ql=2 after objective-function convergence.: "when iterations of Ql=2 is bigger than 74, the increment of objective function is close to 0. Therefore, we choose the result after the 74th training as the default CBOW model of Ql=2"
- [results] CBOW models were built and trained using the Python package Gensim.: "the all CBOW models were built and trained by using the software package of Python called Gensim"
