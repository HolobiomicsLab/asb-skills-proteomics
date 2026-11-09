---
name: msnset-spatial-proteomics-data-handling
description: Use when a quantitative MS-based spatial proteomics dataset (e.g. LOPIT-style
  subcellular fractionation) is held in an MSnSet with a 'markers' feature column,
  and the goal is PCA ordination/visualization of subcellular niches with downstream
  classifier scores.
when_to_use_negative:
- Input is raw peptide identification data (mzID/fasta) to be populated into Pbase
  Proteins objects rather than an already-quantified MSnSet
- The dataset lacks a 'markers' feature column, so there are no labelled subcellular
  niches to color or validate the classification against
- The analysis goal is peptide/protein identification processing rather than visualization
  of a processed expression matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Analysis environment for loading the MSnSet, ordinating intensities via prcomp,
    and rendering the plot
- name: pRoloc
  role: Provides plot2D(method='PCA') for ordination of MSnSet expression data and
    visualization of marker/label columns
- name: pRolocdata
  role: Supplies the annotated Tan et al. (2009) Drosophila embryo MSnSet (tan2009r1)
    with 'markers' and 'svm' feature columns
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:d9544bc166d18ad95a1ea7f37b9ed259482260b4a8caf76cd5cece8d52527125
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pRoloc
  - pRolocdata
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# msnset-spatial-proteomics-data-handling

## Summary

Handling and visualizing quantitative spatial proteomics data stored in MSnSet objects with R/Bioconductor: loading annotated datasets such as the Tan et al. Drosophila embryo set (tan2009r1), exploiting 'markers' and classification-score feature columns, and ordinating protein intensities via PCA with pRoloc's plot2D.

## When to use

Use when a quantitative MS-based spatial proteomics dataset (e.g. LOPIT-style subcellular fractionation) is held in an MSnSet with a 'markers' feature column, and the goal is PCA ordination/visualization of subcellular niches with downstream classifier scores.

## When NOT to use

- Input is raw peptide identification data (mzID/fasta) to be populated into Pbase Proteins objects rather than an already-quantified MSnSet
- The dataset lacks a 'markers' feature column, so there are no labelled subcellular niches to color or validate the classification against
- The analysis goal is peptide/protein identification processing rather than visualization of a processed expression matrix

## Inputs

- MSnSet expression object with quantitative protein intensities (e.g. tan2009r1 from pRolocdata, Tan et al. 2009 Drosophila embryo dataset)
- Feature metadata columns 'markers' (organelle/subcellular niche labels) and 'svm' (per-protein SVM classification scores)

## Outputs

- PCA scatter plot (plot2D) colored by markers across 11 subcellular niches, with point size encoding SVM scores and axes annotated with variance explained by PC1/PC2
- Exported figure file, e.g. figure8_right_pca_svm_size.png, reproducible against Figure 8 (right) of the paper

## How to apply

Load the MSnSet (e.g. tan2009r1 from pRolocdata, carrying 'markers' and 'svm' feature columns) and inspect fData to confirm which feature variables encode organelle labels and classifier confidence. Call pRoloc's plot2D with method='PCA' so protein intensities in the expression matrix are ordinated via prcomp. Map the 'markers' feature column onto point colors to display the 11 subcellular niches, and scale point size (cex) by the 'svm' feature column so per-protein SVM classification confidence is visually encoded. Add a legend and axis labels reporting the proportion of variance explained by the first two principal components, then export the figure (e.g. figure8_right_pca_svm_size.png) for comparison against the published figure.

## Related tools

- **R** (Analysis environment for loading the MSnSet, ordinating intensities via prcomp, and rendering the plot)
- **pRoloc** (Provides plot2D(method='PCA') for ordination of MSnSet expression data and visualization of marker/label columns)
- **pRolocdata** (Supplies the annotated Tan et al. (2009) Drosophila embryo MSnSet (tan2009r1) with 'markers' and 'svm' feature columns)

## Examples

```
library(pRolocdata); data(tan2009r1); plot2D(tan2009r1, fcol = "markers", method = "PCA", cex = tan2009r1$svm)
```

## Evaluation signals

- Axis labels of the plot2D output report the proportion of variance explained by PC1 and PC2, as produced by prcomp on the expression matrix
- The plot shows exactly 11 distinct colored subcellular niches matching the levels of the 'markers' feature column
- Point sizes vary monotonically with the 'svm' feature column values
- Exported PNG is visually consistent with Figure 8 (right) of Gatto et al. 2015 (DOI 10.1002/pmic.201400392)
- fData of the loaded MSnSet contains both 'markers' and 'svm' columns before plotting

## Limitations

- Method is demonstrated on the specific tan2009r1 dataset whose feature columns ('markers', 'svm') are precomputed; other datasets require their own marker labels and classifier scores
- PCA is one of several ordination choices for plot2D; results depend on the expression matrix and may need alternative methods for other spatial proteomics designs
- Bioconductor packages in this ecosystem evolve quickly (the companion Pbase README notes it 'is under heavy development and is likely to considerably change in the near future'), so function interfaces should be re-checked against the installed versions

## Evidence

- [other] Figure 8 (right) visualizes 11 subcellular niches for Drosophila embryos from Tan et al., with SVM classification scores encoded as point size: [evidence span withheld — anchor status: not_found]
- [abstract] Data visualization is an essential tool for data exploration and communication: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [readme] Pbase interoperability with MSnSet instances: [evidence span withheld — anchor status: external_doc]
