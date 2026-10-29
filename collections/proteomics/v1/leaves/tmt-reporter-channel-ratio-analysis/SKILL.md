---
name: tmt-reporter-channel-ratio-analysis
description: Use when you have TMT reporter-ion intensities from a labelled MS experiment
  and need per-feature log2 channel ratios (M) versus mean log2 intensity (A) plotted
  as MA plots, e.g. to visualise ratio spread between reporter channels.
when_to_use_negative:
- The input is already a processed feature table with pre-computed fold-changes and
  no reporter-channel intensities to re-derive M and A from
- The experiment is label-free or uses a different labelling scheme (e.g. SILAC) where
  TMT reporter-ion channel ratios do not apply
- The goal is quantitative statistical testing of channel differences rather than
  qualitative visualisation with R's plotting systems
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Analysis and plotting environment used for all three plotting systems (traditional,
    lattice, ggplot2)
- name: Pbase
  role: Provides the example-data infrastructure and Proteins data object used to
    load the PXD000001 TMT 6-plex reporter-ion data
  repo: github.com/lgatto/Pbase
- name: knitr
  role: Vignette engine used to render the article's reproducible plotting code as
    rmarkdown
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:7f59e8e9a1af3d944df0136f5946166cfcd03cced7eeda63e11a776723e1ad18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pbase
  - R
  - knitr
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tmt-reporter-channel-ratio-analysis

## Summary

Computes per-feature M (log2 intensity ratio of a TMT reporter channel over a reference channel) and A (mean log2 reporter intensity) values from a TMT 6-plex experiment (PXD000001, HeLa/PRTC on a Thermo Orbitrap Q Exactive) and renders MA plots with R's traditional, lattice and ggplot2 plotting systems, as in Figure 1 of Gatto et al. 2015.

## When to use

Use when you have TMT reporter-ion intensities from a labelled MS experiment and need per-feature log2 channel ratios (M) versus mean log2 intensity (A) plotted as MA plots, e.g. to visualise ratio spread between reporter channels.

## When NOT to use

- The input is already a processed feature table with pre-computed fold-changes and no reporter-channel intensities to re-derive M and A from
- The experiment is label-free or uses a different labelling scheme (e.g. SILAC) where TMT reporter-ion channel ratios do not apply
- The goal is quantitative statistical testing of channel differences rather than qualitative visualisation with R's plotting systems

## Inputs

- TMT 6-plex reporter-ion intensity data from the PXD000001 experiment (HeLa plus PRTC, Thermo Orbitrap Q Exactive), accessed via Pbase example-data infrastructure
- Channel pairing specification: TMT129 vs TMT127 and TMT131 vs TMT127

## Outputs

- Per-feature M values (log2 intensity ratio of TMT129 or TMT131 over TMT127)
- Per-feature A values (mean log2 reporter intensity of the compared channels)
- Three MA scatter panels rendered with base R, lattice and ggplot2
- Side-by-side three-panel figure saved as tmt_ma_plots.png

## How to apply

Load the TMT 6-plex reporter-ion data (PXD000001; Gatto et al. HeLa/PRTC run on a Thermo Orbitrap Q Exactive) into R via the Pbase example-data infrastructure. For each feature, compute the M value as the log2 intensity ratio of the TMT129 channel (and separately the TMT131 channel) over the TMT127 reference channel, and the A value as the mean log2 reporter intensity of the two compared channels. Render the first MA plot as a scatter of M versus A with R's traditional/base plotting system, the second with the lattice formula interface (xyplot of M ~ A), and the third with ggplot2's geom_point, using mostly default settings as in the article. Arrange the three panels side by side and save as tmt_ma_plots.png to match the layout of Figure 1; the article's vignette is rendered with knitr/rmarkdown and BiocStyle, so the plotting code is typically executed inside a knitr chunk.

## Related tools

- **R** (Analysis and plotting environment used for all three plotting systems (traditional, lattice, ggplot2))
- **Pbase** (Provides the example-data infrastructure and Proteins data object used to load the PXD000001 TMT 6-plex reporter-ion data) — github.com/lgatto/Pbase
- **knitr** (Vignette engine used to render the article's reproducible plotting code as rmarkdown)

## Examples

```
library(Pbase); data(tmt); M <- log2(assayData(tmt)$tmt129 / assayData(tmt)$tmt127); A <- rowMeans(log2(assayData(tmt)[c('tmt129','tmt127')])); xyplot(M ~ A)
```

## Evaluation signals

- The saved tmt_ma_plots.png contains three rendered MA scatter panels, one per plotting system (base/traditional, lattice, ggplot2)
- Each panel plots M (log2 channel ratio) versus A (mean log2 reporter intensity) for the same underlying per-feature data
- The qualitative spread of points in each panel visually matches Figure 1 of the article, panel by panel (no quantitative threshold is reported in the source)
- The R script executes successfully end to end with mostly default plotting settings
- The three panels are arranged side by side, matching the layout of Figure 1

## Limitations

- No quantitative acceptance threshold is reported in the source; validation is the qualitative panel-by-panel visual comparison against Figure 1 plus script execution success
- Pbase is under heavy development and likely to considerably change in the near future, per the README, so data-loading APIs may shift
- The demonstration uses the Pbase example data (HeLa/PRTC, PXD000001) with UniProt release 2015_02 sequences; applying it to other datasets requires supplying your own reporter-intensity data

## Evidence

- [results] The three MA plots in Figure 1 were produced with the same input data (PXD000001 TMT 6-plex, TMT129/TMT131 vs TMT127) using mostly default settings of the traditional plotting system, lattice and ggplot2.: [evidence span withheld — anchor status: not_found]
- [methods] Compute, per feature, the M value as the log2 intensity ratio of the TMT129 (and separately TMT131) channel over the TMT127 channel and the A value as the mean log2 reporter intensity.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we provided a short overview of the application of the R software to the visualization of proteomics data.: "Here, we provided a short overview of the application of the R software to the visualization of proteomics data."
- [readme] Pbase is under heavy development and is likely to considerably change in the near future.: "`Pbase` is under heavy development and is likely to considerably
change in the near future."
