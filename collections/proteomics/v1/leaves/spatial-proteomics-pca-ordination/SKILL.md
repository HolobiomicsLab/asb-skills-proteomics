---
name: spatial-proteomics-pca-ordination
description: Use when a quantitative spatial proteomics dataset (e.g. an MSnSet such
  as tan2009r1 from pRolocdata) needs 2D ordination for visualising subcellular niches
  and per-protein SVM classification confidence.
when_to_use_negative:
- Input is a Pbase Proteins object or raw peptide identifications (mzID/FASTA) rather
  than a quantitative MSnSet with markers annotation — use the Pbase Proteins workflow
  instead
- The goal is predicting peptide detectability/prototypicity with SVM or other models
  (as in the README's flyability ideas) rather than visualising organelle classification
  on labelled spatial data
- The dataset lacks a 'markers' feature column, so organelle niches cannot be colour-encoded
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_0091
tools:
- name: pRoloc
  role: Provides plot2D (method='PCA', prcomp-based ordination) and SVM classification
    used to build the spatial proteomics map
- name: pRolocdata
  role: Supplies the tan2009r1 MSnSet example dataset with 'markers' and 'svm' feature
    columns
- name: R
  role: Environment in which the PCA ordination and plot2D visualisation are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:66833745543190015c13a8ab3754018af1dd1fccf15c0a4f24dca06d35e73296
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pRoloc
  - pRolocdata
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# spatial-proteomics-pca-ordination

## Summary

Ordinate quantitative spatial proteomics data (an MSnSet expression matrix) by PCA with pRoloc's plot2D, colouring proteins by the 'markers' organelle labels and encoding SVM classification scores as point size, reproducing the Tan et al. Drosophila embryo Figure 8 (right) style of map.

## When to use

Use when a quantitative spatial proteomics dataset (e.g. an MSnSet such as tan2009r1 from pRolocdata) needs 2D ordination for visualising subcellular niches and per-protein SVM classification confidence.

## When NOT to use

- Input is a Pbase Proteins object or raw peptide identifications (mzID/FASTA) rather than a quantitative MSnSet with markers annotation — use the Pbase Proteins workflow instead
- The goal is predicting peptide detectability/prototypicity with SVM or other models (as in the README's flyability ideas) rather than visualising organelle classification on labelled spatial data
- The dataset lacks a 'markers' feature column, so organelle niches cannot be colour-encoded

## Inputs

- MSnSet with quantitative spatial proteomics intensities, e.g. tan2009r1 from pRolocdata (Tan et al. 2009 Drosophila embryo dataset)
- Feature metadata columns 'markers' (organelle/niche labels) and 'svm' (SVM classification scores)

## Outputs

- 2D PCA scatter plot coloured by the 11 subcellular niches with point size encoding SVM scores
- Figure file, e.g. figure8_right_pca_svm_size.png, validated against Figure 8 (right) of the paper

## How to apply

Load the Tan et al. (2009) Drosophila embryo spatial proteomics MSnSet (tan2009r1 from pRolocdata), which carries the 'markers' and 'svm' feature columns. Call pRoloc's plot2D with method='PCA' so protein intensities are ordinated via prcomp on the expression matrix. Colour points by the 'markers' feature column to display the 11 subcellular niches, and scale point size (cex) according to the 'svm' feature column so per-protein SVM classification confidence is encoded visually. Add a legend and axis labels reporting the proportion of variance explained by the first two principal components. This follows the paper's framing that R's plotting systems are used 'to visualize and understand raw and processed MS-based proteomics data'.

## Related tools

- **pRoloc** (Provides plot2D (method='PCA', prcomp-based ordination) and SVM classification used to build the spatial proteomics map)
- **pRolocdata** (Supplies the tan2009r1 MSnSet example dataset with 'markers' and 'svm' feature columns)
- **R** (Environment in which the PCA ordination and plot2D visualisation are executed)

## Examples

```
library(pRoloc); data(tan2009r1, package="pRolocdata"); plot2D(tan2009r1, fcol = "markers", method = "PCA", cex = tan2009r1$svm)
```

## Evaluation signals

- Plot legend lists the 11 subcellular niches defined in the 'markers' feature column
- Axis labels report the proportion of variance explained by the first two principal components from prcomp
- Point sizes vary monotonically with values in the 'svm' feature column
- Saved figure (figure8_right_pca_svm_size.png) visually matches Figure 8 (right) of the Tan et al. paper
- Ordination is computed on the MSnSet expression matrix (prcomp), not on arbitrary feature columns

## Limitations

- SVM-based models do not necessarily transfer: 'a model of one species/dataset could not be transfered to another dataset (without dramatically decreasing the performance)' — so SVM scores on the tan2009r1 data should not be assumed valid for other organisms or instruments
- The curated article text covers R/Bioconductor visualisation broadly (e.g. Pbase Proteins objects); the pRoloc-specific plot2D parameters are grounded in the task card, not the article body

## Evidence

- [abstract] R plotting systems summary: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [abstract] Visualization for exploration: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [readme] SVM model transferability: [evidence span withheld — anchor status: external_doc]
