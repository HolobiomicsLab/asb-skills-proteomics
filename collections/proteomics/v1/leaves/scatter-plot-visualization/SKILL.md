---
name: scatter-plot-visualization
description: Use when you have a high-dimensional, class-labeled feature matrix (e.g.,
  LGBM-selected BiLSTM + UniRep features with labels 1=R protein, 0=non-R protein)
  and need to visually confirm class separability after dimensionality reduction.
when_to_use_negative:
- Input is raw FASTA sequences — sequences must first be embedded (BiLSTM/UniRep/TAPE-BERT)
  and feature-selected before scatter visualization is meaningful
- The goal is quantitative classifier evaluation (accuracy 0.956, F1 0.933, AUC 0.997)
  rather than qualitative cluster inspection — use fivefold cross-validation metrics
  instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: umap-learn
  role: Python package used to project the LGBM-selected high-dimensional features
    into a 2D UMAP embedding for scatter plotting
- name: prPred-DRLF
  role: Source of the plant R protein dataset (Wang et al. [9], 456 sequences) and
    the LGBM-feature-selected feature matrix being visualized
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Feature selection method that produced the reduced-dimensionality feature
    subset whose cluster separation is being visualized
- name: Python 3.7
  role: Execution environment for the prPred-DRLF toolkit and the UMAP plotting script
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:5954d5d234260835fb5ae2b0e39fe7b339ecf72937bd9c11868f774b1367b211
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - LightGBM (LGBM)
  - Python
  - Python 3.7
  - prPred-DRLF
  - umap-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scatter-plot-visualization

## Summary

Generates 2D scatter plots (e.g., UMAP embeddings) colored by class label to visually verify that a feature-selection/embedding pipeline separates protein classes — here, non-R vs R proteins in the prPred-DRLF plant resistance protein workflow (Figure 4).

## When to use

Use when you have a high-dimensional, class-labeled feature matrix (e.g., LGBM-selected BiLSTM + UniRep features with labels 1=R protein, 0=non-R protein) and need to visually confirm class separability after dimensionality reduction.

## When NOT to use

- Input is raw FASTA sequences — sequences must first be embedded (BiLSTM/UniRep/TAPE-BERT) and feature-selected before scatter visualization is meaningful
- The goal is quantitative classifier evaluation (accuracy 0.956, F1 0.933, AUC 0.997) rather than qualitative cluster inspection — use fivefold cross-validation metrics instead

## Inputs

- LGBM-feature-selected feature matrix of the plant R protein dataset (456 sequences: 152 positive, 304 negative) from the prPred-DRLF repository dataset directory
- Binary class label vector (1 = R protein, 0 = non-R protein) aligned with feature matrix rows

## Outputs

- umap_scatter_plot.png — 2D scatter plot of UMAP embedding, points colored by R vs non-R label
- 2D UMAP embedding array (n_samples x 2)

## How to apply

Load the LGBM-feature-selected feature matrix from the prPred-DRLF repository (https://github.com/Wangys-prog/prPred-DRLF), where labels 1 denote R proteins and 0 denote non-R proteins. Apply UMAP dimensionality reduction with the umap-learn Python package to project the selected features into a two-dimensional embedding space. Generate a scatter plot of the 2D UMAP embedding with points colored by class label (R protein vs. non-R protein), mirroring Figure 4 of the article. Inspect the plot to confirm that non-R and R proteins form distinct clusters, then save the figure as umap_scatter_plot.png. The rationale is that the article reports the two-dimensional UMAP space separated the non-R and R proteins into distinct clusters after LGBM feature selection reduced the fused BiLSTM + UniRep vector (5505D) dimensionality.

## Related tools

- **umap-learn** (Python package used to project the LGBM-selected high-dimensional features into a 2D UMAP embedding for scatter plotting)
- **prPred-DRLF** (Source of the plant R protein dataset (Wang et al. [9], 456 sequences) and the LGBM-feature-selected feature matrix being visualized) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Feature selection method that produced the reduced-dimensionality feature subset whose cluster separation is being visualized)
- **Python 3.7** (Execution environment for the prPred-DRLF toolkit and the UMAP plotting script)

## Examples

```
import umap; emb = umap.UMAP(n_components=2).fit_transform(X); plt.scatter(emb[:,0], emb[:,1], c=labels); plt.savefig('umap_scatter_plot.png')
```

## Evaluation signals

- Output figure umap_scatter_plot.png exists and is a valid rendered image
- Scatter plot shows non-R and R proteins in distinct clusters, consistent with Figure 4 of the article
- Each scatter point corresponds one-to-one with a row of the LGBM-selected feature matrix and its class label
- Embedding is 2D (n_samples x 2), matching the 'two-dimensional UMAP space' reported in the article

## Limitations

- UMAP cluster separation is qualitative visual evidence only; it does not substitute for quantitative evaluation (accuracy, F1-score, AUC, recall)
- Deep representation learning feature methods require an elevated level of computing resources; a GPU speeds up the prPred-DRLF pipeline upstream of plotting
- Reproduced cluster separation may differ from Figure 4 due to UMAP's stochastic initialization and hyperparameter sensitivity

## Evidence

- [results] The two-dimensional UMAP space separated the non-R and R proteins into distinct clusters, and feature vector dimensionality was reduced by the LGBM method (Figure 4).: [evidence span withheld — anchor status: not_found]
- [other] Generate a scatter plot of the 2D UMAP embedding with points colored by class label (R protein vs. non-R protein) to reproduce Figure 4.: [evidence span withheld — anchor status: not_found]
