---
name: focal-loss-binary-classification-training
description: Use when a high-dimensional concatenated feature vector (e.g., 1011-D
  word-embedding + bag-of-words GPCR features) must be reduced via a trained DL module
  whose Layer-1 output is the final representation.
when_to_use_negative:
- Input is already a reduced 505-D feature table produced by Layer-1 interception
  — feed it directly to XGBoost instead
- The task is multi-class GPCR family prediction rather than the binary GPCR identification
  setup with focal loss / binary cross-entropy
- CBOW models have not been trained and raw sequences must first be converted to word-embedding
  and BOW features
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Extreme Gradient Boosting
  role: Downstream classifier consuming the 505-D Layer-1 intercepted features produced
    by this training step
  repo: https://github.com/454170054/EMCBOW-GPCR
- name: XGBoost
  role: Selected as the final algorithm because it achieved the highest Acc, Mcc and
    AUC on the reduced features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:e9627ab6f94e514489beb55d97a1665a047c5a1d21b160fe43b116543a8aa001
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
  - XGBoost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Focal-loss binary classification training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a small deep-learning network (three fully connected layers, two Batch Normalization layers, Leaky ReLU activations) on concatenated GPCR feature vectors (1011-D) using focal loss / binary cross-entropy with the Adam optimizer (learning rate 0.01), and intercept the 505-D Layer-1 output as dimension-reduced features for downstream XGBoost.

## When to use

Use when a high-dimensional concatenated feature vector (e.g., 1011-D word-embedding + bag-of-words GPCR features) must be reduced via a trained DL module whose Layer-1 output is the final representation.

## When NOT to use

- Input is already a reduced 505-D feature table produced by Layer-1 interception — feed it directly to XGBoost instead
- The task is multi-class GPCR family prediction rather than the binary GPCR identification setup with focal loss / binary cross-entropy
- CBOW models have not been trained and raw sequences must first be converted to word-embedding and BOW features

## Inputs

- Concatenated 1011-D F_GPCR feature vectors (WEF word-embedding features + BF bag-of-words features) from the EMCBOW-GPCR benchmark dataset
- Class labels for each GPCR sample
- Python 3.7.0 virtual environment with packages from requirements.txt

## Outputs

- Trained DL model (three fully connected layers, two Batch Normalization layers, Leaky ReLU)
- 505-D reduced feature matrix intercepted from Layer 1, saved as dl_features.npy
- Saved model weights reusable by Predict.py

## How to apply

Load the concatenated 1011-D F_GPCR feature vectors (word-embedding features WEF + bag-of-words features BF) with their class labels from the EMCBOW-GPCR benchmark dataset files. Build a DL module with three fully connected layers and two Batch Normalization layers using Leaky ReLU activations, sized so the output of Layer 1 (505-D) can be intercepted as the final feature representation. Compile with focal loss / binary cross-entropy and the Adam optimizer at learning rate 0.01, then train for a maximum of 20 epochs with batch size 32, applying Early Stopping with a patience of 3 epochs to halt when validation performance stops improving. Run the trained network on all samples, intercept the Layer-1 output to obtain the 505-D reduced feature matrix, and save the weights and features (e.g., dl_features.npy) for the downstream Extreme Gradient Boosting classifier. Note that the README requires a fresh Python 3.7.0 virtual environment with the packages in requirements.txt, and GetResults.py must be run first to generate the models.

## Related tools

- **Extreme Gradient Boosting** (Downstream classifier consuming the 505-D Layer-1 intercepted features produced by this training step) — https://github.com/454170054/EMCBOW-GPCR
- **XGBoost** (Selected as the final algorithm because it achieved the highest Acc, Mcc and AUC on the reduced features)

## Examples

```
python GetResults.py
```

## Evaluation signals

- The saved feature matrix has exactly one 505-D vector per sample, matching the paper's Layer-1 interception design
- Training respects the 20-epoch cap with 3-epoch early-stopping patience
- Optimizer settings match Adam with learning rate 0.01 and focal loss / binary cross-entropy
- The reduced features improve downstream XGBoost performance (paper reports combined-feature AUC of 0.9423 before this design)
- Models are generated in a fresh Python 3.7.0 environment with all requirements.txt packages installed before first run

## Limitations

- The DL model structure (number of layers and units per layer) is flexible and difficult to find the best configuration
- The hyperparameter N used in Word Embedding is hard to determine and may significantly influence feature quality
- Version numbers for software packages are only listed in requirements.txt, not in the paper
- First-run models must be generated via GetResults.py before prediction is possible

## Evidence

- [full_text] The trained DL model consumes the 1011-D F_GPCR feature vector, and the output of Layer 1 is intercepted as the final features, which are 505-D after dimensionality reduction.: [evidence span withheld — anchor status: not_found]
