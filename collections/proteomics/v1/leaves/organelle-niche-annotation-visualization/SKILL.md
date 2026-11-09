---
name: organelle-niche-annotation-visualization
description: Use when a quantified spatial proteomics MSnSet (e.g. tan2009r1 from
  pRolocdata) carries 'markers' and 'svm' feature columns and the goal is a 2D PCA
  map of organelle niches with SVM scores visualized.
when_to_use_negative:
- Input is raw LC-MS/MS spectra or peptide identification results (mzID, FASTA) rather
  than a quantified MSnSet — use protein/peptide manipulation workflows (e.g. Pbase
  Proteins objects) instead
- The MSnSet lacks the 'markers' and 'svm' feature columns, so niches cannot be colored
  and SVM confidence cannot be encoded as point size
- The analysis goal is peptide detectability/proteotypicity prediction (e.g. the SVM
  models described in the Pbase README) rather than visualizing organelle classification
  on a PCA map
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pRoloc
  role: Provides plot2D (method='PCA') for ordinating protein intensities and encoding
    'markers' colors and 'svm' score point sizes
- name: pRolocdata
  role: Supplies the tan2009r1 MSnSet (Tan et al. 2009 Drosophila embryo spatial proteomics
    data)
- name: R
  role: Implementation language for the visualization workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:23fd6450b32e52f925077f2b6e521e3ed39d020b9bdfc7b8e4a7c597c80dfb36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pRoloc
  - pRolocdata
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Organelle niche annotation visualization (pRoloc plot2D PCA + SVM score encoding)

## Summary

Reproduces spatial proteomics maps such as Figure 8 (right) of Gatto et al. 2015: an MSnSet of quantitative protein intensities is ordinated by PCA via pRoloc's plot2D, points colored by the 'markers' feature column (11 Drosophila embryo subcellular niches), and per-protein SVM classification confidence encoded as point size.

## When to use

Use when a quantified spatial proteomics MSnSet (e.g. tan2009r1 from pRolocdata) carries 'markers' and 'svm' feature columns and the goal is a 2D PCA map of organelle niches with SVM scores visualized.

## When NOT to use

- Input is raw LC-MS/MS spectra or peptide identification results (mzID, FASTA) rather than a quantified MSnSet — use protein/peptide manipulation workflows (e.g. Pbase Proteins objects) instead
- The MSnSet lacks the 'markers' and 'svm' feature columns, so niches cannot be colored and SVM confidence cannot be encoded as point size
- The analysis goal is peptide detectability/proteotypicity prediction (e.g. the SVM models described in the Pbase README) rather than visualizing organelle classification on a PCA map

## Inputs

- MSnSet with quantitative protein intensities and 'markers' feature column (organelle niche labels)
- MSnSet with 'svm' feature column holding per-protein SVM classification scores
- tan2009r1 from pRolocdata (Tan et al. 2009 Drosophila embryo dataset, 11 subcellular niches)

## Outputs

- 2D PCA ordination plot (plot2D) colored by organelle niche with legend
- Point sizes scaled by SVM classification scores
- Axis labels reporting proportion of variance explained by PC1 and PC2
- PNG file, e.g. figure8_right_pca_svm_size.png

## How to apply

Load the Tan et al. (2009) Drosophila embryo spatial proteomics MSnSet, tan2009r1, from pRolocdata; it must contain the 'markers' and 'svm' feature columns. Call pRoloc's plot2D with method='PCA' so protein intensities are ordinated via prcomp on the expression matrix. Color points by the 'markers' feature column to display the 11 subcellular niches, and scale point size (cex) by the 'svm' feature column so SVM classification confidence is encoded visually. Add a legend and axis labels reporting the proportion of variance explained by the first two principal components. Save the plot (e.g. figure8_right_pca_svm_size.png) and validate against Figure 8 (right) of the paper. Rationale: PCA on the expression matrix is the ordination used in the article's R/Bioconductor visualization workflow for processed MS-based proteomics data.

## Related tools

- **pRoloc** (Provides plot2D (method='PCA') for ordinating protein intensities and encoding 'markers' colors and 'svm' score point sizes)
- **pRolocdata** (Supplies the tan2009r1 MSnSet (Tan et al. 2009 Drosophila embryo spatial proteomics data))
- **R** (Implementation language for the visualization workflow)

## Examples

```
library(pRoloc); data(tan2009r1, package='pRolocdata'); plot2D(tan2009r1, fcol='markers', cex=tan2009r1$svm)
```

## Evaluation signals

- Legend lists exactly the 11 subcellular niches of the Tan et al. Drosophila embryo dataset
- PC1/PC2 axis labels report the proportion of variance explained by prcomp on the expression matrix
- Point size (cex) varies monotonically with the 'svm' feature column values per protein
- Saved PNG (figure8_right_pca_svm_size.png) matches Figure 8 (right) of the paper on visual inspection
- Point colors are consistent with the 'markers' feature column assignments

## Limitations

- Encoding SVM scores as point size requires the 'svm' feature column to already exist in the MSnSet; the visualization does not itself perform SVM classification
- Only the first two principal components are displayed, so variance not captured by PC1/PC2 is not shown
- The companion R/Bioconductor packages evolve rapidly — the Pbase README notes the ecosystem 'is under heavy development and is likely to considerably change in the near future', so function signatures should be checked

## Evidence

- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [other] plot2D PCA ordination and SVM score encoding: [evidence span withheld — anchor status: not_found]
- [other] SVM scores as point size (task card finding): [evidence span withheld — anchor status: not_found]
- [readme] MSnSet interoperability in the R proteomics ecosystem: "protein identification results (`mzID` or (devel) `mzR` packages) and possibly
also `MSnExp` and `MSnSet` instances."
