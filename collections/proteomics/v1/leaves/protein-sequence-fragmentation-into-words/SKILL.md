---
name: protein-sequence-fragmentation-into-words
description: Use when raw FASTA protein sequences must be converted into token units
  for NLP-based feature extraction (CBOW word embedding or BOW wordbook features)
  prior to classifier training.
when_to_use_negative:
- Input is already a numeric feature matrix (e.g. precomputed WEF or BF vectors) —
  fragmentation is a raw-sequence preprocessing step
- The target task requires residue-level or structural features rather than fixed-length
  sequence-level NLP features
- Sequences contain many non-standard residues beyond 'X' that the AAindex mapping
  does not handle
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: Gensim
  role: Python package used to build and train the CBOW models on the fragment wordbooks
    after splitting
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- name: EMCBOW-GPCR
  role: Source repository providing the benchmark GPCR sequences and the fragmentation/feature-extraction
    code
  repo: https://github.com/454170054/EMCBOW-GPCR
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:5c541f53bc0891ca464c515417e6ba92d2c1769646a46246c60c069790d34cca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - EMCBOW-GPCR
  - Gensim
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# protein-sequence-fragmentation-into-words

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Splits GPCR amino-acid primary sequences into fixed-length fragment 'words' (overlapping k-mers of length l=2, 3, and 4) so that NLP techniques — Word2vec CBOW word embedding and AAindex-based bag-of-words (BOW) with wordbooks — can be applied to protein sequences.

## When to use

Use when raw FASTA protein sequences must be converted into token units for NLP-based feature extraction (CBOW word embedding or BOW wordbook features) prior to classifier training.

## When NOT to use

- Input is already a numeric feature matrix (e.g. precomputed WEF or BF vectors) — fragmentation is a raw-sequence preprocessing step
- The target task requires residue-level or structural features rather than fixed-length sequence-level NLP features
- Sequences contain many non-standard residues beyond 'X' that the AAindex mapping does not handle

## Inputs

- GPCR protein sequences in FASTA format from the benchmark dataset (EMCBOW-GPCR repository)
- Fragment lengths l=2, l=3, l=4 (word sizes defining the three wordbooks)

## Outputs

- Wordbooks Ql=2, Ql=3, Ql=4 built from the split fragments
- Per-sequence word-embedding features EF2, EF3, EF4 concatenated into an 896-D WEF vector (wef_features.csv)
- Per-sequence 115-dimensional AAindex-based BOW feature vector BF

## How to apply

Load the GPCR benchmark sequences from the EMCBOW-GPCR repository and split each sequence into overlapping words of fragment length l=2, l=3, and l=4; these three word lengths define the three wordbooks used downstream. For the word-embedding branch, the fragments build wordbooks Ql=2, Ql=3, and Ql=4 that train Gensim CBOW models (Negative Sampling), sweeping hidden vector size N over {128, 256, 512} and selecting the default model by objective-function convergence (74th iteration for l=2, 100th for l=3, 280th for l=4). For the BOW branch, the same fragments are encoded with AAindex physicochemical indices (mapping 'X' to the AAindex mean across standard amino acids), and the wordbook size is chosen by maximizing the weighted Silhouette Coefficient. Fragment occurrences are then counted against the wordbooks to produce the 115-D BOW feature vector BF, while per-sequence fragment features EF2, EF3, EF4 are concatenated into the 896-D WEF vector via formula (11).

## Related tools

- **Gensim** (Python package used to build and train the CBOW models on the fragment wordbooks after splitting) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- **EMCBOW-GPCR** (Source repository providing the benchmark GPCR sequences and the fragmentation/feature-extraction code) — https://github.com/454170054/EMCBOW-GPCR

## Examples

```
python Predict.py "test.fasta"
```

## Evaluation signals

- Every sequence yields a 115-dimensional BOW feature vector BF; the feature matrix row count equals the number of benchmark sequences
- EF2, EF3, and EF4 concatenate into a single 896-D WEF vector per sequence (formula 11), written to wef_features.csv
- CBOW training convergence matches the paper's checkpoints: l=2 at the 74th iteration, l=3 at the 100th, l=4 at the 280th (objective-function increment close to 0)
- Downstream sanity reference: BOW features alone reach AUC = 0.9344 and combined features AUC = 0.9423 with CatBoost under 5-fold cross-validation (20 repeats)
- Wordbook sizes for the BOW branch are justified by maximizing the weighted Silhouette Coefficient

## Limitations

- The word-embedding hyperparameter N (hidden vector size) is flexible and hard to determine, and may significantly influence feature quality
- Version numbers of Gensim and other packages are only available indirectly via the repository requirements.txt, not stated in the paper
- README requires a dedicated Python 3.7.0 virtual environment with requirements.txt packages; models must be generated by running GetResults.py before prediction
- The ambiguous residue 'X' must be mapped to the AAindex mean; other non-standard residues are not discussed

## Evidence

- [other] The BOW feature-extraction pipeline encodes GPCR sequences with AAindex (mapping 'X' to the AAindex mean): [evidence span withheld — anchor status: not_found]
- [other] The word-embedding features of each GPCR sequence (EF2, EF3, EF4 from fragment lengths l=2, 3, 4) are combined into an 896-D WEF vector via formula (11): [evidence span withheld — anchor status: not_found]
