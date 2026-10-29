---
name: silhouette-based-wordbook-size-selection
description: Use when converting GPCR (or other protein) sequences into bag-of-words
  features and the number of words in the wordbook must be determined from the data
  via the weighted Silhouette Coefficient.
when_to_use_negative:
- The input is already a numeric feature table or embedding (e.g. CBOW word-embedding
  vectors) rather than raw protein sequences needing BOW encoding
- The goal is word-embedding features (word vectors from the CBOW models trained with
  Gensim) rather than count-based BOW wordbook features
- Sequences contain characters that cannot be encoded by AAindex and are not covered
  by the 'X'-to-mean mapping rule
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:222d7b465a965c1727898fb874c1376969c0d387e216cdcdd5defdd0dbbd2b9f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  tools: []
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# silhouette-based-wordbook-size-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the wordbook size for BOW feature extraction over protein sequences by clustering AAindex-encoded sequence fragments and choosing the size that maximizes the weighted Silhouette Coefficient, yielding the 115-dimensional BF feature vector used by EMCBOW-GPCR.

## When to use

Use when converting GPCR (or other protein) sequences into bag-of-words features and the number of words in the wordbook must be determined from the data via the weighted Silhouette Coefficient.

## When NOT to use

- The input is already a numeric feature table or embedding (e.g. CBOW word-embedding vectors) rather than raw protein sequences needing BOW encoding
- The goal is word-embedding features (word vectors from the CBOW models trained with Gensim) rather than count-based BOW wordbook features
- Sequences contain characters that cannot be encoded by AAindex and are not covered by the 'X'-to-mean mapping rule

## Inputs

- GPCR protein sequences in FASTA format from the EMCBOW-GPCR benchmark dataset (https://github.com/454170054/EMCBOW-GPCR)
- AAindex physicochemical index values for the 20 standard amino acids (with 'X' mapped to the index mean)
- Candidate wordbook sizes (fragment-cluster counts) to evaluate

## Outputs

- A wordbook: the set of clustered fragment words at the Silhouette-optimal size
- A 115-dimensional BOW feature vector BF per input sequence (feature matrix with one row per benchmark sequence)

## How to apply

Encode each residue of the GPCR sequence numerically using AAindex physicochemical indices, mapping the ambiguous residue 'X' to the mean of the index across the standard amino acids. Split each encoded sequence into fixed-length fragments (words) and cluster the fragments across candidate wordbook sizes. Compute the weighted Silhouette Coefficient for each candidate size and select the wordbook size that maximizes it. Count fragment occurrences of each sequence against the chosen wordbook to produce the 115-dimensional BOW feature vector BF per sequence. Validate that every sequence yields a well-formed 115-D row and that the resulting BOW-feature classifier reaches performance comparable to the paper-reported AUC = 0.9344 for BOW features.

## Examples

```
conda create -n emcbow python=3.7.0; pip install -r requirements.txt; python GetResults.py
```

## Evaluation signals

- Every input sequence yields exactly a 115-dimensional BOW feature vector; no shorter or longer rows
- Feature-matrix row count equals the number of benchmark dataset sequences and contains no missing values
- The chosen wordbook size corresponds to the maximum weighted Silhouette Coefficient across all candidate sizes evaluated
- A classifier trained on the BF features (e.g. with CatBoost/XGBoost in the EMCBOW-GPCR pipeline) approaches the paper-reported BOW-feature AUC of 0.9344
- The 'X' residues in input sequences are encoded to the AAindex mean rather than dropped or erroring out

## Limitations

- The wordbook size (and related hyperparameter N in word embedding) is flexible and hard to determine the best value, and may significantly influence feature quality
- The specific BOW feature-extraction process is only referred to reference [46] in the paper, so exact fragmentation parameters may need recovery from the repository code
- Reproducing the pipeline requires a Python 3.7.0 virtual environment with packages pinned in the repository's requirements.txt; tool version numbers are not stated in the paper itself

## Evidence

- [methods] Cluster the fragments and determine the wordbook size by maximizing the weighted Silhouette Coefficient.: [evidence span withheld — anchor status: not_found]
