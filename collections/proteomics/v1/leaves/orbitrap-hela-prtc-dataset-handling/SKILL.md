---
name: orbitrap-hela-prtc-dataset-handling
description: Use when a task requires reproducing plots or analyses from the PXD000001
  TMT 6-plex HeLa/PRTC Orbitrap Q Exactive experiment in R, or needs the Pbase Proteins
  example data (fasta + mzid) as a test dataset.
when_to_use_negative:
- The input is already a processed feature/quantification table from a different instrument
  or labeling scheme (the skill is tied to the TMT 6-plex reporter-ion structure of
  this Orbitrap Q Exactive run).
- The goal is raw spectral or PSM-level visualization rather than reporter-ion ratio
  (MA-style) comparison of TMT channels.
- A quantitative reproduction with numeric acceptance thresholds is required — the
  source only supports qualitative panel-by-panel visual comparison.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Analysis environment; provides the three plotting systems (traditional, lattice,
    ggplot2) used to render the MA panels
- name: Pbase
  role: Supplies the Proteins data object and example-data infrastructure for the
    HeLa/PRTC Orbitrap Q Exactive dataset
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: mzID
  role: Reads the Thermo_Hela_PRTC_selected.mzid identification file used to populate
    the Proteins object
- name: knitr
  role: Vignette engine used to render the reproducible visualization workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:59da7aa91f9a51e4da2f4c198fb337c6b4e69f9815bf00896c93f3a6d5ab3215
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pbase
  - R
  - knitr
  - mzID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# orbitrap-hela-prtc-dataset-handling

## Summary

Loading and manipulating the HeLa/PRTC LC-MSMS dataset acquired on a Thermo Orbitrap Q Exactive (PXD000001 TMT 6-plex) in R via the Pbase package's Proteins infrastructure, to support downstream visualization such as the three-system MA plots.

## When to use

Use when a task requires reproducing plots or analyses from the PXD000001 TMT 6-plex HeLa/PRTC Orbitrap Q Exactive experiment in R, or needs the Pbase Proteins example data (fasta + mzid) as a test dataset.

## When NOT to use

- The input is already a processed feature/quantification table from a different instrument or labeling scheme (the skill is tied to the TMT 6-plex reporter-ion structure of this Orbitrap Q Exactive run).
- The goal is raw spectral or PSM-level visualization rather than reporter-ion ratio (MA-style) comparison of TMT channels.
- A quantitative reproduction with numeric acceptance thresholds is required — the source only supports qualitative panel-by-panel visual comparison.

## Inputs

- PXD000001 TMT 6-plex reporter-ion intensity data (HeLa/PRTC, Thermo Orbitrap Q Exactive) via Pbase example data
- HUMAN_2015_02_selected.fasta (restricted set of high-scoring human proteins, UniProt release 2015_02)
- Thermo_Hela_PRTC_selected.mzid (peptide identification results from MSGF+)

## Outputs

- tmt_ma_plots.png containing three side-by-side MA scatter panels (base, lattice, ggplot2)
- Per-feature M (log2 TMT129/TMT127 and TMT131/TMT127) and A (mean log2 reporter intensity) values
- Pbase Proteins object populated with protein sequences and mapped peptide identifications

## How to apply

Load the TMT 6-plex reporter-ion data from the PXD000001 experiment (HeLa/PRTC run on a Thermo Orbitrap Q Exactive) using Pbase's example-data infrastructure, which pairs HUMAN_2015_02_selected.fasta with Thermo_Hela_PRTC_selected.mzid (identifications searched with MSGF+ against UniProt 2015_02). For the Figure 1 MA plots, compute per-feature M values as the log2 intensity ratio of the TMT129 (and separately TMT131) channel over TMT127, and A values as the mean log2 reporter intensity of the two compared channels. Draw the same M-versus-A scatter three times using R's three plotting systems: base/traditional plot(), lattice xyplot(M ~ A), and ggplot2 geom_point, mostly with default settings. Arrange the panels side by side and save as tmt_ma_plots.png to match the article's Figure 1 layout.

## Related tools

- **R** (Analysis environment; provides the three plotting systems (traditional, lattice, ggplot2) used to render the MA panels)
- **Pbase** (Supplies the Proteins data object and example-data infrastructure for the HeLa/PRTC Orbitrap Q Exactive dataset) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **mzID** (Reads the Thermo_Hela_PRTC_selected.mzid identification file used to populate the Proteins object)
- **knitr** (Vignette engine used to render the reproducible visualization workflow)

## Examples

```
library("Pbase"); fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase"); idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package = "Pbase")
```

## Evaluation signals

- The saved figure contains three rendered MA scatter panels corresponding to base, lattice and ggplot2
- Each panel's qualitative spread visually matches the corresponding panel of Figure 1 of the article
- The R script executes successfully end-to-end against the Pbase example data (fasta + mzid)
- M values center near 0 with symmetric spread, consistent with log2 reporter-intensity ratios of compared channels
- The Proteins object is populated from both the fasta (sequences) and mzid (peptides) inputs without errors

## Limitations

- Acceptance is a qualitative visual check — no quantitative threshold is reported in the source.
- The underlying instrument data are limited to this specific HeLa/PRTC Orbitrap Q Exactive TMT 6-plex acquisition; extension to other datasets is not validated.
- Pbase is under heavy development and its interface is likely to change; README recommends filing issues for major contributions.

## Evidence

- [intro] Instrument and dataset context for the example data: "acquired on a Thermo Orbitrap Q
Exactive instrument."
- [methods] Pbase is the package providing the central data infrastructure: "**Package:** [`Pbase`](http://bioconductor.org/packages/devel/bioc/html/Pbase.html)"
- [intro] Identification file used to populate the Proteins object: "idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid",
                      package = "Pbase")"
- [readme] Source dataset is searched identifications, per the README: "protein identification results (`mzID` or (devel) `mzR` packages) and possibly
also `MSnExp` and `MSnSet` instances."
