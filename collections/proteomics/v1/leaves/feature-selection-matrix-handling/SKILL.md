---
name: feature-selection-matrix-handling
description: Use when a labeled protein feature matrix (e.g., 5505D fused BiLSTM+UniRep
  vectors for R vs non-R plant proteins) is too high-dimensional for direct classification
  and feature importance-based dimensionality reduction is needed.
when_to_use_negative:
- Input is already a low-dimensional or curated feature table where feature selection
  would discard meaningful dimensions
- The task is raw sequence embedding rather than selection — use the deep representation
  learning encoders (BiLSTM, UniRep, TAPE-BERT) first; this skill operates on the
  resulting numerical vectors
- Sequences are unredundant-unfiltered and contain duplicates — apply CD-HIT with
  a 30% identity cutoff first to avoid leakage into train/test splits
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LightGBM (LGBM)
  role: Feature selection via feature importance ranking and downstream LGBM classification
    of the selected subset
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred-DRLF
  role: Python toolkit providing the feature matrices, selection scripts, and LGBM
    prediction pipeline (Python 3.7, lightgbm-2.3.0)
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: CD-HIT
  role: Preprocessing step to remove redundant sequences at a 30% identity cutoff
    before feature extraction and selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:1dfc4f7fa567d6692099f2bc63e3bc83acf556a43235592e9b66a16dbb705798
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - CD-HIT
  - LightGBM (LGBM)
  - Python
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# feature-selection-matrix-handling

## Summary

Select an optimal feature subset from a high-dimensional fused deep representation learning feature matrix (BiLSTM 3605D + UniRep 1900D = 5505D) for plant R protein identification, using LGBM (or RF/MRMD3.0) feature importance ranking, and handle the reduced matrix for downstream classification and UMAP visualization.

## When to use

Use when a labeled protein feature matrix (e.g., 5505D fused BiLSTM+UniRep vectors for R vs non-R plant proteins) is too high-dimensional for direct classification and feature importance-based dimensionality reduction is needed.

## When NOT to use

- Input is already a low-dimensional or curated feature table where feature selection would discard meaningful dimensions
- The task is raw sequence embedding rather than selection — use the deep representation learning encoders (BiLSTM, UniRep, TAPE-BERT) first; this skill operates on the resulting numerical vectors
- Sequences are unredundant-unfiltered and contain duplicates — apply CD-HIT with a 30% identity cutoff first to avoid leakage into train/test splits

## Inputs

- Labeled fused BiLSTM + UniRep feature matrix (5505D) for plant R proteins, labels 1 = R protein, 0 = non-R protein (from https://github.com/Wangys-prog/prPred-DRLF/tree/master/dataset)
- FASTA protein sequences (456 sequences: 152 positive, 304 negative) for embedding via BiLSTM/UniRep/TAPE-BERT through eFeature
- Alternative feature matrices (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D) if re-running selection from individual embedding types

## Outputs

- LGBM-selected (or RF/MRMD-selected) reduced feature matrix from the original 5505D fused vectors
- Feature importance rankings from RF, LGBM, and MRMD3.0
- Trained LGBM classifier predictions (R protein vs non-R protein) with accuracy, F1-score, precision, recall, and AUC
- 2D UMAP scatter plot of the selected features (umap_scatter_plot.png) showing cluster separation

## How to apply

Load the fused BiLSTM + UniRep feature matrix (3605D + 1900D = 5505D) with binary labels (1 = R protein, 0 = non-R protein) from the prPred-DRLF repository dataset (Wang et al. plant R protein dataset, 456 sequences after CD-HIT 30% identity cutoff). Apply LGBM feature selection — optionally compared against RF and MRMD3.0 — ranking features by their importance values to obtain an optimal feature subset, which reduces dimensionality. Feed the selected feature subset into classifiers (SVM, LGBM, RF, ET) with fivefold cross-validation, and optionally project the LGBM-selected features into 2D UMAP space to visually confirm that non-R and R proteins form distinct clusters. The rationale is that deep representation learning embeddings are high-dimensional and feature selection identifies the most informative subset before classification.

## Related tools

- **LightGBM (LGBM)** (Feature selection via feature importance ranking and downstream LGBM classification of the selected subset) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred-DRLF** (Python toolkit providing the feature matrices, selection scripts, and LGBM prediction pipeline (Python 3.7, lightgbm-2.3.0)) — https://github.com/Wangys-prog/prPred-DRLF
- **CD-HIT** (Preprocessing step to remove redundant sequences at a 30% identity cutoff before feature extraction and selection)

## Examples

```
conda create -n prPred-DRLF python=3.7 && conda activate prPred-DRLF && pip3 install -r requirements.txt && python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Selected feature dimensionality is strictly less than the original 5505D fused BiLSTM+UniRep dimension, confirming reduction occurred
- Fivefold cross-validation metrics (accuracy, F1-score, AUC) on the selected subset are comparable to or better than the full fused feature set
- Independent test performance in the reported range (accuracy 0.956, F1-score 0.933, AUC 0.997)
- 2D UMAP of LGBM-selected features visually separates non-R and R proteins into distinct clusters (Figure 4 consistency check)
- Feature importance rankings are produced for all three compared methods (RF, LGBM, MRMD3.0) to justify the chosen subset

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources
- Parallel computing is not yet incorporated into the program; high computational efficiency is deferred to future work
- LGBM 2.3.0 and Python 3.7 with specific pinned dependencies (numpy 1.19.2, pandas 1.2.0, torch 1.2.0) are required per the README; version mismatches may break the pipeline

## Evidence

- [intro] Three feature selection techniques, including random forest (RF), LGBM, and max-relevance-max-distance (MRMD), were applied to select an optimal feature subset from the fused BiLSTM + Uni Rep feature: [evidence span withheld — anchor status: not_found]
- [results] The two-dimensional UMAP space separated the non-R and R proteins into distinct clusters, and feature vector dimensionality was reduced by the LGBM method (Figure 4).: [evidence span withheld — anchor status: not_found]
