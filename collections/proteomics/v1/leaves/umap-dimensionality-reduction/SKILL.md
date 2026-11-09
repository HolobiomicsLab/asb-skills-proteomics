---
name: umap-dimensionality-reduction
description: Use when you need a 2D visual check that a selected feature subset (e.g.,
  LGBM-ranked features from fused BiLSTM 3605D + UniRep 1900D embeddings) separates
  two protein classes before or after classifier training.
when_to_use_negative:
- You need a quantitative performance metric rather than a qualitative visual check
  — use the LGBM classifier with fivefold cross-validation and report accuracy, F1-score,
  and AUC instead.
- The input is still the raw fused 5505D feature vector before feature selection;
  apply RF/LGBM/MRMD feature selection first, as the reported separation is of the
  LGBM-selected features.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: prPred-DRLF
  role: Source of the LGBM-feature-selected feature matrix and plant R protein dataset
    used as UMAP input
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Feature selection method that produced the reduced feature subset visualized
    by UMAP
- name: Python 3.7
  role: Environment in which the UMAP projection and scatter plotting are executed
    (prPred-DRLF is a Python-based toolkit)
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:6eb95ac3ce40197967fe07e352a883e6307b935b312be2e6eb5d64ffc04ab1da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - LightGBM (LGBM)
  - Python
  - Python 3.7
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

# umap-dimensionality-reduction

## Summary

Applies UMAP to project the LGBM-feature-selected BiLSTM + UniRep feature matrix of the plant R protein dataset into a two-dimensional embedding, producing a scatter plot that visually verifies whether non-R and R proteins separate into distinct clusters (Figure 4 of prPred-DRLF).

## When to use

Use when you need a 2D visual check that a selected feature subset (e.g., LGBM-ranked features from fused BiLSTM 3605D + UniRep 1900D embeddings) separates two protein classes before or after classifier training.

## When NOT to use

- You need a quantitative performance metric rather than a qualitative visual check — use the LGBM classifier with fivefold cross-validation and report accuracy, F1-score, and AUC instead.
- The input is still the raw fused 5505D feature vector before feature selection; apply RF/LGBM/MRMD feature selection first, as the reported separation is of the LGBM-selected features.

## Inputs

- LGBM-feature-selected feature matrix (rows = protein sequences from the 456-sequence plant R protein dataset, columns = selected BiLSTM + UniRep features)
- Binary class label vector (1 = R protein, 0 = non-R protein; 152 positive, 304 negative)

## Outputs

- Two-dimensional UMAP embedding of the selected feature matrix (n_samples x 2 coordinates)
- Scatter plot colored by class label saved as umap_scatter_plot.png

## How to apply

Load the LGBM-feature-selected feature matrix from the prPred-DRLF repository (https://github.com/Wangys-prog/prPred-DRLF), where the plant R protein dataset (Wang et al. dataset, 456 sequences: 152 positive R proteins, 304 negative non-R proteins) carries labels 1 = R protein and 0 = non-R protein. Apply UMAP via the umap-learn Python package to reduce the high-dimensional selected features to a two-dimensional embedding, keeping the class labels attached to each row. Generate a scatter plot of the 2D UMAP coordinates with points colored by class label, mirroring Figure 4 of the article. Inspect the plot to confirm that non-R and R proteins form distinct clusters; save the figure as umap_scatter_plot.png. The rationale is that LGBM feature selection reduced the fused 5505D BiLSTM + UniRep vector to an optimal subset, and cluster separation in UMAP space provides qualitative evidence that the retained features carry discriminative information.

## Related tools

- **prPred-DRLF** (Source of the LGBM-feature-selected feature matrix and plant R protein dataset used as UMAP input) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Feature selection method that produced the reduced feature subset visualized by UMAP)
- **Python 3.7** (Environment in which the UMAP projection and scatter plotting are executed (prPred-DRLF is a Python-based toolkit)) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python -c "import umap, pandas as pd; X = pd.read_csv('lgbm_selected_features.csv', index_col=0); emb = umap.UMAP(n_components=2).fit_transform(X); pd.DataFrame(emb, columns=['UMAP1','UMAP2']).to_csv('umap_embedding.csv')"
```

## Evaluation signals

- umap_scatter_plot.png exists in the output directory.
- The scatter plot visually shows non-R and R proteins forming distinct clusters, consistent with Figure 4 of the article.
- The number of points in the plot equals the dataset size (456 sequences: 152 R, 304 non-R).
- The UMAP embedding has exactly two dimensions per sample.
- Point colors correspond one-to-one with the binary labels (1 = R protein, 0 = non-R protein) in the input matrix.

## Limitations

- UMAP cluster separation is a qualitative visual signal, not a substitute for the quantitative evaluation (accuracy, F1-score, AUC) used in the article.
- Deep representation learning features require elevated computing resources; a GPU speeds up the upstream embedding steps but is not generated by UMAP itself.
- The result depends on the upstream LGBM feature selection; running UMAP on unselected 5505D features may not reproduce the reported separation.

## Evidence

- [results] The two-dimensional UMAP space separated the non-R and R proteins into distinct clusters, and feature vector dimensionality was reduced by the LGBM method (Figure 4).: [evidence span withheld — anchor status: not_found]
- [abstract] The Python script can be downloaded from https://github.com/Wangys-prog/pr Pred-DRLF.: [evidence span withheld — anchor status: not_found]
