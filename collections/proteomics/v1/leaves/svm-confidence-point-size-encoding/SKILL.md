---
name: svm-confidence-point-size-encoding
description: Use when visualizing a spatial proteomics MSnSet on a PCA ordination
  and you need to communicate classifier confidence alongside organelle assignments,
  e.g. after pRoloc SVM classification with per-feature scores.
when_to_use_negative:
- Data is raw spectra or PSM identifications rather than a quantitative MSnSet with
  organelle marker annotations
- No trained classifier scores (e.g. 'svm' feature column) are available, so point
  size encoding would be arbitrary
- Goal is peptide/protein sequence manipulation with Pbase Proteins objects rather
  than spatial map visualization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Runtime environment for the visualization workflow described in the paper's
    overview of R plotting systems
- name: pRoloc
  role: Provides plot2D PCA visualization and the spatial proteomics classification
    framework whose SVM scores are encoded as point size
- name: pRolocdata
  role: Source of the tan2009r1 Drosophila embryo MSnSet with 'markers' and 'svm'
    feature columns
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:30afc22c68f8add71802461345e82c4a3e2dff5a3beb71afad6c64cc85ad5a93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pRoloc
  - pRolocdata
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# svm-confidence-point-size-encoding

## Summary

Encodes per-protein SVM organelle classification confidence as point size (cex) on a pRoloc plot2D PCA map of quantitative spatial proteomics data, reproducing Figure 8 (right) for the Tan et al. Drosophila embryo dataset with 11 subcellular niches.

## When to use

Use when visualizing a spatial proteomics MSnSet on a PCA ordination and you need to communicate classifier confidence alongside organelle assignments, e.g. after pRoloc SVM classification with per-feature scores.

## When NOT to use

- Data is raw spectra or PSM identifications rather than a quantitative MSnSet with organelle marker annotations
- No trained classifier scores (e.g. 'svm' feature column) are available, so point size encoding would be arbitrary
- Goal is peptide/protein sequence manipulation with Pbase Proteins objects rather than spatial map visualization

## Inputs

- MSnSet object with quantitative spatial proteomics intensities (tan2009r1 from pRolocdata)
- Feature column 'markers' with organelle labels for 11 subcellular niches
- Feature column 'svm' with per-protein SVM classification scores

## Outputs

- PCA scatter plot (figure8_right_pca_svm_size.png) colored by markers and sized by SVM scores
- Axis labels reporting proportion of variance explained by PC1 and PC2

## How to apply

Load the Tan et al. (2009) Drosophila embryo MSnSet (tan2009r1 from pRolocdata), which carries the 'markers' and 'svm' feature columns. Call pRoloc's plot2D with method='PCA' so protein intensities are ordinated via prcomp on the expression matrix, coloring points by the 'markers' feature column to display the 11 subcellular niches. Scale point size (cex) according to the 'svm' feature column so per-protein SVM confidence is encoded visually. Add a legend and axis labels reporting the proportion of variance explained by the first two principal components, then save as figure8_right_pca_svm_size.png and validate against Figure 8 (right) of the paper.

## Related tools

- **R** (Runtime environment for the visualization workflow described in the paper's overview of R plotting systems)
- **pRoloc** (Provides plot2D PCA visualization and the spatial proteomics classification framework whose SVM scores are encoded as point size)
- **pRolocdata** (Source of the tan2009r1 Drosophila embryo MSnSet with 'markers' and 'svm' feature columns)

## Examples

```
library(pRoloc); data(tan2009r1, package='pRolocdata'); plot2D(tan2009r1, fcol='markers', cex=2*tan2009r1$svm)
```

## Evaluation signals

- Output PNG matches Figure 8 (right) of Gatto et al. 2015 in layout, colors, and point-size gradient
- Plot displays exactly 11 subcellular niches for the Drosophila embryo dataset
- Axis labels report the proportion of variance explained by the first two principal components from prcomp on the expression matrix
- Point sizes vary monotonically with the 'svm' feature column values in the MSnSet

## Limitations

- PCA ordination only captures variance along the first two principal components; structure outside PC1/PC2 is not represented
- Requires an existing per-feature classifier score column — confidence encoding is not produced by the plot itself
- Paper focuses on R/Bioconductor visualization; porting to other plotting systems may lose the cex-based size encoding

## Evidence

- [methods] workflow source card: [evidence span withheld — anchor status: not_found]
- [abstract] article abstract: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [readme] Pbase README interoperability: "The package allows to easily interact with `AAString` and
`AAStringSet` instances, protein databases such as UniProt"
