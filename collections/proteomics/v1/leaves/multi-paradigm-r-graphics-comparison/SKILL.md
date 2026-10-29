---
name: multi-paradigm-r-graphics-comparison
description: Use when comparing or demonstrating R's base, lattice and ggplot2 graphics
  paradigms on the same proteomics quantitative data, e.g. reproducing MA plots from
  PXD000001 TMT 6-plex reporter ions (TMT129/TMT131 vs TMT127).
when_to_use_negative:
- The input is already a summarized feature table of M and A values from a different
  instrument/labeling scheme, where the article's PXD000001 TMT reporter-ion example
  does not apply.
- The goal is a publication-quality single-panel figure in one graphics system rather
  than a cross-paradigm comparison of R's three plotting systems.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: R
  role: language in which all three plotting systems (base, lattice, ggplot2) are
    exercised on the same MA data
- name: Pbase
  role: provides the central Proteins data object and example-data infrastructure
    for loading the PXD000001 TMT reporter-ion data
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: knitr
  role: vignette engine used to render the reproducible rmarkdown document containing
    the plots
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:c61181b2d7645e2e4606b06206a8dcaf7ca669df606253f8c07ebe84955857da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pbase
  - R
  - knitr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# multi-paradigm-r-graphics-comparison

## Summary

Reproduce a single proteomics visualization (MA scatter plots of TMT reporter-ion log2 ratios versus mean log2 intensity) three times side by side, using R's traditional/base plotting system, the lattice formula interface, and ggplot2, to illustrate and compare the three plotting systems described by Gatto et al.

## When to use

Use when comparing or demonstrating R's base, lattice and ggplot2 graphics paradigms on the same proteomics quantitative data, e.g. reproducing MA plots from PXD000001 TMT 6-plex reporter ions (TMT129/TMT131 vs TMT127).

## When NOT to use

- The input is already a summarized feature table of M and A values from a different instrument/labeling scheme, where the article's PXD000001 TMT reporter-ion example does not apply.
- The goal is a publication-quality single-panel figure in one graphics system rather than a cross-paradigm comparison of R's three plotting systems.

## Inputs

- PXD000001 TMT 6-plex reporter-ion intensities (TMT127, TMT129, TMT131 channels) from Pbase example data
- HeLa/PRTC peptide identification data from Thermo Orbitrap Q Exactive acquisition

## Outputs

- tmt_ma_plots.png: a figure with three side-by-side MA scatter panels (base graphics, lattice, ggplot2)

## How to apply

Load the TMT 6-plex reporter-ion data from the PXD000001 experiment (HeLa/PRTC run on a Thermo Orbitrap Q Exactive) using the Pbase example-data infrastructure. Per feature, compute M as the log2 intensity ratio of TMT129 (and separately TMT131) over TMT127, and A as the mean log2 reporter intensity of the two compared channels. Render three MA scatter panels from the same input using mostly default settings: (1) R's traditional/base plotting system, (2) lattice via the formula interface xyplot(M ~ A), and (3) ggplot2 with geom_point. Arrange the panels side by side and save as tmt_ma_plots.png to match the layout of Figure 1 in the article.

## Related tools

- **R** (language in which all three plotting systems (base, lattice, ggplot2) are exercised on the same MA data)
- **Pbase** (provides the central Proteins data object and example-data infrastructure for loading the PXD000001 TMT reporter-ion data) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **knitr** (vignette engine used to render the reproducible rmarkdown document containing the plots)

## Examples

```
xyplot(M ~ A, data = tmt)
```

## Evaluation signals

- The output PNG contains exactly three rendered MA scatter panels corresponding to base, lattice and ggplot2.
- All three panels use the same input data (M = log2 TMT129/TMT127 and log2 TMT131/TMT127; A = mean log2 reporter intensity), so their qualitative point-cloud spread should visually match Figure 1 of the article panel by panel.
- The R script executes without error, including the lattice xyplot(M ~ A) call and the ggplot2 geom_point call.
- The three panels were produced with mostly default settings of each plotting system, as stated in the source workflow.

## Limitations

- The source reports no quantitative acceptance threshold: acceptance is the qualitative panel-by-panel visual check plus script execution success.
- Pbase is explicitly under heavy development and likely to considerably change, so the example-data infrastructure may shift between versions.
- The comparison covers only R's three standard plotting systems with default settings; specialized or highly tuned graphics are out of scope.

## Evidence

- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [intro] acquired on a Thermo Orbitrap Q Exactive instrument: "acquired on a Thermo Orbitrap Q Exactive instrument."
- [intro] Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration"
- [intro] The vignette briefly introduces the central data object of the `Pbase` package, namely `Proteins` instances: "This vignette briefly introduces the central data object of the `Pbase` package, namely `Proteins` instances"
- [readme] Pbase is under heavy development and is likely to considerably change in the near future.: [evidence span withheld — anchor status: external_doc]
