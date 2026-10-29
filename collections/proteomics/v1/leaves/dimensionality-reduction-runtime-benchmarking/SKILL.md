---
name: dimensionality-reduction-runtime-benchmarking
description: Use when a large pairwise similarity or distance matrix (e.g., TM-Align
  structural similarity scores) must be embedded in 2D for cluster visualization and
  a choice must be made between UMAP and t-SNE on speed and projection quality.
when_to_use_negative:
- The input is raw protein sequences or predicted structures rather than an already-computed
  pairwise similarity matrix — TM-Align must be run first to produce tm-matrix.
- The goal is 3D structure prediction or clustering itself (use AlphaFold2/ColabFold
  and the clustering notebook) rather than benchmarking 2D projection methods on an
  existing matrix.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: UMAP
  role: Primary dimensionality reduction method applied to the TM-Align similarity
    matrix to produce the 2D structural cluster embedding
- name: t-SNE
  role: Alternative dimensionality reduction method run on the same matrix as the
    benchmark comparison against UMAP
- name: TM-Align
  role: Source of the pairwise protein structural similarity matrix that serves as
    the embedding input
- name: ColabFold
  role: AlphaFold2 derivative whose clustering notebook workflow supplies the structural
    cluster memberships used to color the projections
  repo: https://github.com/sokrypton/ColabFold
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:77b5fdc51bb19c4bafc714a5fb778019a5ffb17dc13f762f8e1419f9066e0d87
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/KeaunAmani/cbp60db
  repo_url: https://github.com/sokrypton/ColabFold
  tools:
  - ColabFold
  - TM-Align
  - UMAP
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# dimensionality-reduction-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark UMAP against t-SNE for 2D projection of the TM-Align pairwise protein structural similarity matrix (~1996 unique CBP60 proteins), recording wall-clock runtime and projection comparability for each method into a metrics table.

## When to use

Use when a large pairwise similarity or distance matrix (e.g., TM-Align structural similarity scores) must be embedded in 2D for cluster visualization and a choice must be made between UMAP and t-SNE on speed and projection quality.

## When NOT to use

- The input is raw protein sequences or predicted structures rather than an already-computed pairwise similarity matrix — TM-Align must be run first to produce tm-matrix.
- The goal is 3D structure prediction or clustering itself (use AlphaFold2/ColabFold and the clustering notebook) rather than benchmarking 2D projection methods on an existing matrix.

## Inputs

- TM-Align structural similarity matrix (tm-matrix., extracted from tm-matrix..tar.bz2) for ~1996 unique CBP60 proteins
- Structural cluster membership labels from the paper's Google Colab protein clustering notebook

## Outputs

- 2D UMAP embedding of the CBP60 structural similarity matrix
- 2D t-SNE embedding of the same matrix
- Side-by-side scatter plots colored by structural cluster membership
- projection_comparison_metrics.csv (runtime in seconds and projection quality per method)

## How to apply

Load the TM-Align structural similarity matrix (tm-matrix., extracted from tm-matrix..tar.bz2 in the KeaunAmani/cbp60db repository) representing the ~1996 unique CBP60 proteins. Run UMAP on the similarity matrix to produce a 2D embedding, then run t-SNE on the same matrix, recording wall-clock runtime for both. Render both embeddings as side-by-side scatter plots colored by structural cluster membership, following the clustering approach in the paper's Google Colab protein clustering notebook. Tabulate runtime (seconds) and projection quality per method into projection_comparison_metrics.csv, then verify that both projections comparably preserve kingdom-wide structural cluster structure while UMAP completes in substantially less wall-clock time.

## Related tools

- **UMAP** (Primary dimensionality reduction method applied to the TM-Align similarity matrix to produce the 2D structural cluster embedding)
- **t-SNE** (Alternative dimensionality reduction method run on the same matrix as the benchmark comparison against UMAP)
- **TM-Align** (Source of the pairwise protein structural similarity matrix that serves as the embedding input)
- **ColabFold** (AlphaFold2 derivative whose clustering notebook workflow supplies the structural cluster memberships used to color the projections) — https://github.com/sokrypton/ColabFold

## Examples

```
tar -xf tm-matrix..tar.bz2; python -c "import umap, sklearn.manifold; e_umap=umap.UMAP(n_components=2).fit_transform(M); e_tsne=sklearn.manifold.TSNE(n_components=2).fit_transform(M)"
```

## Evaluation signals

- UMAP and t-SNE projections show comparable preservation of the kingdom-wide structural cluster structure when colored by cluster membership
- UMAP completes in substantially less wall-clock time than t-SNE on the same tm-matrix. input
- Both methods' runtime (seconds) and projection quality metrics are present in projection_comparison_metrics.csv
- The embedding covers all ~1996 unique CBP60 proteins from the extracted TM-Align matrix (no rows dropped)

## Limitations

- Projection quality is partly subjective: the authors note UMAP 'generally produces more intuitive projections' in their opinion, so visual comparison should be treated as qualitative as well as metric-based.
- Runtimes are wall-clock measurements and are hardware-dependent; comparisons are only valid when both methods run on the same machine and input matrix.

## Evidence

- [results] Swapping UMAP with t-SNE produces comparable projections, but UMAP is significantly faster and, in the authors' opinion, generally produces more intuitive projections.: [evidence span withheld — anchor status: not_found]
- [methods] extract the TM-Align matrix: "# extract the TM-Align matrix
tar -xf tm-matrix..tar.bz2"
- [abstract] comprehensively characterized 1052 CBP60 gene homologs (encoding 2376 unique transcripts and 1996 unique proteins): "comprehensively characterized 1052 CBP60 gene homologs (encoding 2376 unique transcripts and 1996 unique proteins) across 62 phylogenetically diverse genomes in the plant kingdom"
- [readme] Link to Google Colab notebook for protein clustering: "- [Link to Google Colab notebook for protein clustering](https://colab.research.google.com/drive/1LOZY33CSO5-PdJAdDApyPlfxUu4DHjcW)"
