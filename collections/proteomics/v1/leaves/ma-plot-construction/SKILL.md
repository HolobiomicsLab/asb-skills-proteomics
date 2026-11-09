---
name: ma-plot-construction
description: Use when the goal is to visualize log2 reporter-intensity ratios between
  two TMT channels (e.g. TMT129 or TMT131 vs TMT127) as MA scatter plots, especially
  to contrast R's base, lattice and ggplot2 plotting systems on the same data.
when_to_use_negative:
- Input is already a summary figure or feature table without per-feature reporter-ion
  intensities, so per-feature log2 ratios M and averages A cannot be computed
- The task requires a single quantitative-accuracy criterion against the published
  figure — the source only defines a qualitative visual check
- Channels other than the TMT129/TMT131 vs TMT127 comparisons of PXD000001 are needed
  without recomputing M and A from the relevant reporter-ion columns
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Analysis and plotting environment; its three plotting systems (traditional,
    lattice, ggplot2) each render one MA panel of Figure 1
- name: Pbase
  role: Provides the example-data infrastructure to load the PXD000001 TMT 6-plex
    reporter-ion data and the Proteins data object used in the vignette
  repo: https://github.com/lgatto/Pbase
- name: knitr
  role: Vignette engine (knitr::rmarkdown) used to weave the figure-producing R code
    into the reproducible document
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:1182decffc52056273b32aa99c7d56bdb3482d52f3504645bed46e108c8cfef9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lgatto/Pbase
  tools:
  - Pbase
  - R
  - knitr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ma-plot-construction

## Summary

Reproduces the MA plots of Figure 1 from Gatto et al. (PROTEOMICS, 2015), where the PXD000001 TMT 6-plex reporter-ion data (HeLa/PRTC run on a Thermo Orbitrap Q Exactive) is plotted as M (log2 fold change) versus A (average log2 intensity) three times — once with each of R's three plotting systems (traditional/base, lattice, ggplot2) — to compare their syntax and output on identical proteomics data.

## When to use

Use when the goal is to visualize log2 reporter-intensity ratios between two TMT channels (e.g. TMT129 or TMT131 vs TMT127) as MA scatter plots, especially to contrast R's base, lattice and ggplot2 plotting systems on the same data.

## When NOT to use

- Input is already a summary figure or feature table without per-feature reporter-ion intensities, so per-feature log2 ratios M and averages A cannot be computed
- The task requires a single quantitative-accuracy criterion against the published figure — the source only defines a qualitative visual check
- Channels other than the TMT129/TMT131 vs TMT127 comparisons of PXD000001 are needed without recomputing M and A from the relevant reporter-ion columns

## Inputs

- PXD000001 TMT 6-plex reporter-ion intensity data (Pbase/MSnSet-style features with channels TMT127, TMT129, TMT131)
- R session with the traditional graphics, lattice and ggplot2 packages available

## Outputs

- Three MA scatter panels (M versus A) rendered with base graphics, lattice and ggplot2
- Side-by-side figure saved as tmt_ma_plots.png

## How to apply

Load the PXD000001 TMT 6-plex reporter-ion data into R using the Pbase example-data infrastructure (the same HeLa/PRTC Orbitrap Q Exactive experiment used elsewhere in the vignette). Per feature, compute M as the log2 intensity ratio of the TMT129 channel (and separately TMT131) over the TMT127 channel, and A as the mean log2 reporter intensity of the two compared channels. Render three MA scatter panels of M versus A: (1) plot(M, A) with R's traditional/base system, (2) lattice::xyplot(M ~ A) via the formula interface, and (3) ggplot2 with geom_point — using mostly default settings, as in the article. Arrange the three panels side by side (e.g. with a knitr/rmarkdown vignette engine) and save as tmt_ma_plots.png. Acceptance is script execution success plus a qualitative panel-by-panel visual match to Figure 1 of the article; no quantitative threshold is reported.

## Related tools

- **R** (Analysis and plotting environment; its three plotting systems (traditional, lattice, ggplot2) each render one MA panel of Figure 1)
- **Pbase** (Provides the example-data infrastructure to load the PXD000001 TMT 6-plex reporter-ion data and the Proteins data object used in the vignette) — https://github.com/lgatto/Pbase
- **knitr** (Vignette engine (knitr::rmarkdown) used to weave the figure-producing R code into the reproducible document)

## Examples

```
A <- (lx127 + lx129)/2; M <- lx129 - lx127; plot(A, M); lattice::xyplot(M ~ A); ggplot2::qplot(A, M, geom = "point")
```

## Evaluation signals

- The saved tmt_ma_plots.png contains exactly three rendered MA scatter panels, one per plotting system (base, lattice, ggplot2)
- All three panels are computed from the same input data (TMT129 and TMT131 vs TMT127 reporter intensities of PXD000001) with mostly default plotting settings
- M values are log2 intensity ratios (TMT129/TMT127 and TMT131/TMT127) and A values are the mean log2 intensities of the compared channels, per feature
- Each panel's qualitative spread visually matches the corresponding panel of Figure 1 in the article
- The R script executes without errors inside the knitr/rmarkdown vignette workflow

## Limitations

- No quantitative agreement threshold with Figure 1 is reported in the source; acceptance is only a qualitative panel-by-panel visual check
- Pbase was under heavy development and likely to change considerably (per its README), so the exact data-loading API may differ between versions
- The demonstration compares plotting systems rather than validating quantitative results of the TMT experiment itself

## Evidence

- [abstract] Data visualization plays a key role in high-throughput biology: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [abstract] R overview for proteomics visualization: "Here, we provided a short overview of the application of the R software to the visualization of proteomics data."
- [readme] Pbase README purpose: "Manipulating and exploring protein and proteomics data."
- [readme] Pbase interoperability: [evidence span withheld — anchor status: external_doc]
