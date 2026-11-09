---
name: benchmark-method-comparison
description: Use when a proposed classifier must be compared head-to-head with a published
  baseline on the same benchmark dataset and negative-sample segmentation, and mean
  AUC/Acc/Precision must be assembled into a comparison table.
when_to_use_negative:
- The input is already an assembled feature table — this skill governs the full encode-classify-evaluate
  comparison, not metric computation on precomputed features
- The task is a single-method performance assessment without a published baseline
  to benchmark against (e.g. internal ablation of CBOW vs BOW vs combined features)
- The evaluation requires a different negative-sample strategy than Yu's 4-group segmentation
  — mixing strategies invalidates the comparison
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: XGBoost
  role: Final classifier for the proposed method pipeline; selected because its Acc,
    Mcc and AUC were the highest among tested algorithms
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: Gensim
  role: Python package used to build and train the CBOW word-embedding models whose
    features feed the proposed pipeline being benchmarked
  repo: https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- name: Extreme Gradient Boosting
  role: Receives the dimension-reduced features from the DL model in the proposed
    pipeline during benchmark comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:d7c21e02d331f29a8fc2db52ded601ad84f23677cfb5b7de7194f6059cb024c5
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
  - Extreme Gradient Boosting
  - Gensim
  - XGBoost
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# benchmark-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a like-for-like comparison between a proposed predictor and a published baseline under the same data segmentation, using identical metrics (AUC, Accuracy, Precision) averaged across negative-sample groups. Grounded in the EMCBOW-GPCR Table 4 evaluation against the 400D + PC-PseAAC baseline under Yu's segmentation strategy.

## When to use

Use when a proposed classifier must be compared head-to-head with a published baseline on the same benchmark dataset and negative-sample segmentation, and mean AUC/Acc/Precision must be assembled into a comparison table.

## When NOT to use

- The input is already an assembled feature table — this skill governs the full encode-classify-evaluate comparison, not metric computation on precomputed features
- The task is a single-method performance assessment without a published baseline to benchmark against (e.g. internal ablation of CBOW vs BOW vs combined features)
- The evaluation requires a different negative-sample strategy than Yu's 4-group segmentation — mixing strategies invalidates the comparison

## Inputs

- GPCR benchmark dataset from the EMCBOW-GPCR repository (positive set + 4 groups of 2495 negative sequences per Yu segmentation)
- FASTA-formatted GPCR sequences for feature encoding
- Baseline configuration: 400D PC-PseAAC mixed features + MRMD feature selection + Random Forest

## Outputs

- Per-dataset AUC, Accuracy, and Precision for both methods from 5-fold cross-validation
- Mean AUC/Acc/Precision across the 4 negative-sample groups for each method
- Comparison table (proposed: 95.29/88.11/89.28; baseline: 94.13/86.28/86.62)

## How to apply

Load the GPCR benchmark dataset from the EMCBOW-GPCR repository and construct four evaluation datasets per Yu's segmentation strategy, pairing the positive set with each of the 4 groups of 2495 negative sequences. For each dataset, run the proposed pipeline (CBOW/BOW feature extraction via Gensim, DL dimensionality reduction, XGBoost classifier) and collect AUC, Accuracy, and Precision from 5-fold cross-validation. For each dataset, run the baseline pipeline: encode sequences as 400D PC-PseAAC mixed features, apply MRMD feature selection, and classify with Random Forest, collecting the same three metrics. Average the three metrics across the 4 negative-sample groups for both methods, assemble a comparison table, and verify the means reproduce the reported values (proposed 95.29/88.11/89.28; baseline 94.13/86.28/86.62) within a small tolerance.

## Related tools

- **XGBoost** (Final classifier for the proposed method pipeline; selected because its Acc, Mcc and AUC were the highest among tested algorithms) — https://github.com/454170054/EMCBOW-GPCR
- **Gensim** (Python package used to build and train the CBOW word-embedding models whose features feed the proposed pipeline being benchmarked) — https://github.com/454170054/EMCBOW-GPCR/blob/main/code/GetCBOWFeatures.py
- **Extreme Gradient Boosting** (Receives the dimension-reduced features from the DL model in the proposed pipeline during benchmark comparison)

## Examples

```
python GetResults.py && python Predict.py "test.fasta"
```

## Evaluation signals

- Averaged AUC for the proposed method matches 95.29 and for the 400D + PC-PseAAC baseline matches 94.13 within a small tolerance across the 4 Yu negative-sample groups
- Averaged Accuracy matches 88.11 (proposed) and 86.28 (baseline) within tolerance
- Averaged Precision matches 89.28 (proposed) and 86.62 (baseline) within tolerance
- Each of the 4 evaluation datasets pairs the full positive set with exactly one group of 2495 negative sequences, and all 4 groups are consumed exactly once
- Both methods are evaluated with the same measurement protocol (5-fold cross-validation) on identical data splits

## Limitations

- Software version numbers (Gensim, XGBoost, etc.) are not stated in the paper; they must be taken from the repository requirements.txt, and the README specifies a Python 3.7.0 virtual environment
- The CBOW hyperparameter N is flexible and hard to determine, and may significantly influence feature quality, affecting reproducibility of the compared scores
- The DL model structure (number of layers and units per layer) is flexible and difficult to optimize, so reproduced metrics may drift from reported values
- Models must be generated before prediction: the README instructs running GetResults.py on first use

## Evidence

- [results] Under Yu's segmentation strategy, EMCBOW-GPCR achieved AUC 95.29, Acc 88.11, and Precision 89.28: [evidence span withheld — anchor status: not_found]
- [results] XGBoost selected as final classifier: "the Acc, Mcc and AUC value of XGBoost is the highest. Therefore, XGBoost algorithm is selected as the final algorithm to build classifier in this work"
- [results] CBOW models trained with Gensim: "the all CBOW models were built and trained by using the software package of Python called Gensim"
- [results] 5-fold cross-validation protocol: [evidence span withheld — anchor status: not_found]
- [readme] Python 3.7.0 environment and GetResults.py first-run requirement: "please create a new python virtual environment(python version == 3.7.0) and install packages listed in requirements.txt."
- [discussion] Hyperparameter N limitation: "the hyperparameter N which is used in Word Embedding is flexible and hard to determine the best value. What’s more, it may be influenced the quality of features significantly"
