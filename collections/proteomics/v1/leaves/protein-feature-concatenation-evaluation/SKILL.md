---
name: protein-feature-concatenation-evaluation
description: Use when comparing alternative feature representations (e.g., bag-of-words
  vs word embeddings vs their concatenation) for protein sequence classification with
  a gradient-boosting classifier and cross-validated AUC as the selection criterion.
when_to_use_negative:
- Input is already a single fixed feature table and no representation comparison is
  being made — run the classifier directly instead.
- The goal is final GPCR prediction on a new FASTA file (use Predict.py) rather than
  evaluating feature-representation choices.
- The task requires the final published predictor (which uses XGBoost on DL-reduced
  features), not the CatBoost-based feature-representation comparison.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CatBoost
  role: Default classifier used to compute cross-validated AUC for each feature representation
    in GetResults.py
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetResults.py
- name: Gensim
  role: Trains the CBOW word-embedding models whose features are compared against
    BOW and concatenated features
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- name: XGBoost
  role: Extreme Gradient Boosting algorithm used for the final classifier after feature
    reduction in the full EMCBOW-GPCR pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:54574963ddbee20fcaf4344169a047734e21844806d401bccd4078dadbc7b21b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - CatBoost
  - Gensim
  - XGBoost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# protein-feature-concatenation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates whether concatenating BOW and CBOW word-embedding features of GPCR sequences outperforms either representation alone, by training CatBoost under repeated 5-fold cross-validation and comparing mean AUC (combined 0.9423 vs CBOW 0.9391 vs BOW 0.9344).

## When to use

Use when comparing alternative feature representations (e.g., bag-of-words vs word embeddings vs their concatenation) for protein sequence classification with a gradient-boosting classifier and cross-validated AUC as the selection criterion.

## When NOT to use

- Input is already a single fixed feature table and no representation comparison is being made — run the classifier directly instead.
- The goal is final GPCR prediction on a new FASTA file (use Predict.py) rather than evaluating feature-representation choices.
- The task requires the final published predictor (which uses XGBoost on DL-reduced features), not the CatBoost-based feature-representation comparison.

## Inputs

- Benchmark GPCR sequence dataset with label file (FASTA sequences + class labels)
- CBOW word-embedding features generated per sequence via GetCBOWFeatures.py (Gensim)
- BOW features extracted over split sequence segments
- Concatenated BOW+CBOW feature vectors

## Outputs

- Mean AUC per feature representation (BOW-only, CBOW-only, combined) under repeated 5-fold CV
- Results table (e.g., auc_results.csv) recording AUC values per representation
- Selected default feature representation (combined, AUC = 0.9423) for the final classifier

## How to apply

Split GPCR protein sequences into segments (e.g., Ql=2), train CBOW word-embedding models with Gensim using the repository's GetCBOWFeatures.py and select the converged iteration (74th training for Ql=2, 100th for Ql=3, 280th for Ql=4), and extract BOW features over the split segments. Concatenate the BOW and CBOW feature vectors to form the combined representation. Train the CatBoost classifier — the default algorithm in the repository's GetResults.py — on BOW-only, CBOW-only, and combined features using 5-fold cross-validation repeated 20 times on the benchmark dataset. Select the representation with the highest mean AUC; here combined features (AUC 0.9423) outperformed CBOW (0.9391) and BOW (0.9344) and became the default. Record mean AUC per representation (e.g., in auc_results.csv) to document the comparison.

## Related tools

- **CatBoost** (Default classifier used to compute cross-validated AUC for each feature representation in GetResults.py) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetResults.py
- **Gensim** (Trains the CBOW word-embedding models whose features are compared against BOW and concatenated features) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- **XGBoost** (Extreme Gradient Boosting algorithm used for the final classifier after feature reduction in the full EMCBOW-GPCR pipeline)

## Examples

```
python GetResults.py
```

## Evaluation signals

- Reproduced mean AUC for combined features equals 0.9423 (small tolerance), CBOW = 0.9391, BOW = 0.9344 with CatBoost.
- Ordering invariant holds: combined AUC > CBOW AUC > BOW AUC across all cross-validation repeats.
- Evaluation uses 5-fold cross-validation repeated 20 times on the benchmark dataset for all three feature sets.
- CBOW models were trained with Gensim and iteration selection matches reported convergence points (74th training for Ql=2, 100th for Ql=3, 280th for Ql=4).
- Pipeline runs in a Python 3.7.0 virtual environment with the packages pinned in the repository's requirements.txt.

## Limitations

- Version numbers for Gensim, CatBoost, and XGBoost are not stated in the article; only requirements.txt in the repository specifies the software packages.
- The word-embedding hyperparameter N is flexible, hard to determine, and may significantly influence feature quality.
- The reported comparison is specific to the benchmark GPCR dataset and the CatBoost classifier; the final published method instead feeds features through a DL model and XGBoost.

## Evidence
