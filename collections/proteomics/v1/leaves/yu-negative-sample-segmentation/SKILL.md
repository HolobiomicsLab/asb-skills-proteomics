---
name: yu-negative-sample-segmentation
description: Use when benchmarking a GPCR-identification predictor against the 400D
  + PC-PseAAC mixed-feature baseline of Yu et al. under Yu's segmentation strategy,
  where negatives are divided into 4 groups of 2495 sequences.
when_to_use_negative:
- The evaluation target is a negative-sample strategy other than Yu's 4-group-of-2495
  segmentation — the reported comparison values are only valid under that specific
  split
- The task is training a new predictor from scratch rather than reproducing the two-method
  comparison (use the full EMCBOW-GPCR workflow skill instead)
- Input is already a reduced feature table — this skill assumes raw FASTA GPCR sequences
  and its own feature-extraction pipelines (CBOW/BOW vs 400D PC-PseAAC)
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3307
tools:
- name: XGBoost
  role: Final classifier for the proposed method, selected because its Acc, Mcc and
    AUC were highest among tested algorithms; trained on DL-dimensionality-reduced
    CBOW/BOW features per dataset
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: Gensim
  role: Python package used to build and train the CBOW word-embedding models on segmented
    GPCR sequences before feature concatenation
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:053983437a91a18742f754287c013227df5db1af678b1ae04ced9c98c6f94273
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  repo_url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - Gensim
  - XGBoost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# yu-negative-sample-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the Table 4 benchmark of the EMCBOW-GPCR paper, in which the proposed CBOW/BOW + DL + XGBoost pipeline is compared against the Yu et al. 400D PC-PseAAC + MRMD + Random Forest baseline using Yu's negative-sample segmentation (negatives randomly split into 4 groups of 2495 sequences).

## When to use

Use when benchmarking a GPCR-identification predictor against the 400D + PC-PseAAC mixed-feature baseline of Yu et al. under Yu's segmentation strategy, where negatives are divided into 4 groups of 2495 sequences.

## When NOT to use

- The evaluation target is a negative-sample strategy other than Yu's 4-group-of-2495 segmentation — the reported comparison values are only valid under that specific split
- The task is training a new predictor from scratch rather than reproducing the two-method comparison (use the full EMCBOW-GPCR workflow skill instead)
- Input is already a reduced feature table — this skill assumes raw FASTA GPCR sequences and its own feature-extraction pipelines (CBOW/BOW vs 400D PC-PseAAC)

## Inputs

- GPCR benchmark dataset (FASTA-format sequences) from the EMCBOW-GPCR repository
- Positive GPCR sequence set
- Negative sample set randomly divided into 4 groups of 2495 sequences (Yu segmentation strategy)
- requirements.txt listing Python 3.7.0 package dependencies

## Outputs

- Mean AUC, Accuracy, and Precision per method averaged over the 4 negative-sample groups
- A Table 4-style comparison table (proposed: AUC 95.29/Acc 88.11/Precision 89.28; baseline: AUC 94.13/Acc 86.28/Precision 86.62)
- Trained XGBoost predictor and 400D PC-PseAAC + Random Forest baseline models per dataset

## How to apply

Load the GPCR benchmark dataset from the EMCBOW-GPCR repository and build 4 evaluation datasets, each pairing the positive set with one of the 4 groups of 2495 negative sequences per Yu's segmentation. For each dataset, run the proposed pipeline: split sequences into segments, train CBOW models with Gensim (selecting the training iteration where the objective function increment approaches 0), concatenate BOW and word-embedding features, reduce dimension with a DL model, and classify with XGBoost, collecting AUC, Accuracy, and Precision from 5-fold cross-validation with 20 repetitions. Run the baseline pipeline in parallel: encode sequences as 400D PC-PseAAC mixed features, apply MRMD feature selection, and classify with Random Forest, collecting the same three metrics. Average AUC/Acc/Precision across the 4 negative-sample groups for both methods and assemble a comparison table. Correct execution reproduces the reported means: proposed method AUC 95.29, Acc 88.11, Precision 89.28; baseline AUC 94.13, Acc 86.28, Precision 86.62.

## Related tools

- **XGBoost** (Final classifier for the proposed method, selected because its Acc, Mcc and AUC were highest among tested algorithms; trained on DL-dimensionality-reduced CBOW/BOW features per dataset) — https://github.com/454170054/EMCBOW-GPCR
- **Gensim** (Python package used to build and train the CBOW word-embedding models on segmented GPCR sequences before feature concatenation) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py

## Examples

```
python Predict.py "test.fasta"
```

## Evaluation signals

- Averaged proposed-method metrics across the 4 Yu negative-sample groups match AUC 95.29, Acc 88.11, Precision 89.28 within a small tolerance
- Averaged baseline (400D + PC-PseAAC + RF) metrics match AUC 94.13, Acc 86.28, Precision 86.62 within a small tolerance
- Exactly 4 evaluation datasets are constructed, each containing the positive set plus one group of 2495 negative sequences
- Both methods are evaluated with the same performance measurements (AUC, Accuracy, Precision) from 5-fold cross-validation with 20 repetitions on the same data
- Environment check: Python virtual environment with python version == 3.7.0 and all packages in requirements.txt installed before running the pipeline

## Limitations

- Version numbers for the software tools (e.g., Gensim, CatBoost, XGBoost) are not stated in the paper; only requirements.txt is provided
- The word-embedding hyperparameter N is flexible, hard to determine the best value, and may significantly influence feature quality
- The DL model structure (number of layers and units per layer) is flexible and difficult to optimize, which can affect dimensionality-reduced features fed to XGBoost
- CBOW model selection depends on convergence of the objective function (e.g., 74 iterations for Ql=2), introducing a stopping-criterion sensitivity

## Evidence

- [other] Under Yu's segmentation strategy, EMCBOW-GPCR achieved AUC 95.29, Acc 88.11, and Precision 89.28, all higher than the 400D + PC-PseAAC baseline's AUC 94.13, Acc 86.28, and Precision 86.62.: [evidence span withheld — anchor status: not_found]
- [results] we used two feature extraction methods, i.e., BOW features and Word Embedding features, to represent GPCRs sequences ... 5-fold cross-validation with 20 times on benchmark dataset: [evidence span withheld — anchor status: not_found]
