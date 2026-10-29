---
name: proteomics-data-visualization-r
description: Use when the input is per-pixel MALDI imaging MassSpectrum objects (one
  spectrum per pixel over a tissue section), or LC-MSMS/protein data in R, and the
  goal is visual summarization — mean spectra, ion-image slices, or Proteins/peptide
  plots.
when_to_use_negative:
- The input is already a summarized feature or intensity table with no per-pixel m/z
  dimension to average or slice into ion images
- The task is manipulating protein/identification data itself (populating or exploring
  Proteins instances) rather than visualizing spectra or peptide mappings
- A non-R environment is required — the article's scope is explicitly the application
  of the R software and Bioconductor packages
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: R
  role: Analysis and visualization environment; the paper provides an overview of
    R's plotting systems for raw and processed MS-based proteomics data
- name: MALDIquant
  role: Import per-pixel MassSpectrum objects from the MALDI imaging dataset, average
    spectra across pixels, and pick/rank peaks on the mean spectrum
- name: Cardinal
  role: Cross-check the rendering of the MALDI imaging ion-image slices
- name: Pbase
  role: Provides the central Proteins data object linking protein sequences (FASTA)
    to peptide ranges-of-interest from LC-MSMS/mzID results for visualization
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: mzID
  role: Reads peptide identification results used to populate Proteins instances
- name: MSGF+
  role: Search engine that produced the peptide identifications demonstrated in the
    vignette
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:ec8df5be0ff541d95875757f4e023194056a828f142dec4e06a5815f0cee540d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MALDIquant
  - MSGF+
  - Pbase
  - R
  - mzID
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# proteomics-data-visualization-r

## Summary

Visualize mass-spectrometry and proteomics data in R/Bioconductor, from per-pixel MALDI imaging spectra (mean spectrum plus narrow m/z ion-image slices) to raw and processed proteomics data via packages such as MALDIquant, Cardinal, and Pbase. The paper argues visualization is essential for exploring data structure and patterns of interest in high-throughput biology.

## When to use

Use when the input is per-pixel MALDI imaging MassSpectrum objects (one spectrum per pixel over a tissue section), or LC-MSMS/protein data in R, and the goal is visual summarization — mean spectra, ion-image slices, or Proteins/peptide plots.

## When NOT to use

- The input is already a summarized feature or intensity table with no per-pixel m/z dimension to average or slice into ion images
- The task is manipulating protein/identification data itself (populating or exploring Proteins instances) rather than visualizing spectra or peptide mappings
- A non-R environment is required — the article's scope is explicitly the application of the R software and Bioconductor packages

## Inputs

- Per-pixel MALDI imaging MassSpectrum objects (e.g. the mouse kidney MALDI imaging demonstration dataset)
- Protein sequences from a FASTA file (e.g. HUMAN_2015_02_selected.fasta via Pbase)
- Peptide identification results from LC-MSMS searches (mzID file, e.g. Thermo_Hela_PRTC_selected.mzid; MSGF+ search engine output)
- Data acquired on a Thermo Orbitrap Q Exactive instrument

## Outputs

- A single mean spectrum averaged across all imaging pixels
- Three 2-Da ion-image slices (±1 Da) centered on the three most intense peaks, plotted over the tissue section
- Plots of raw and processed MS-based proteomics data (protein/peptide ranges-of-interest, domain and peptide mappings) using R's plotting systems

## How to apply

For an MS imaging dataset, import per-pixel MassSpectrum objects with MALDIquant's import functions, then collapse all pixels into a single mean spectrum using MALDIquant's spectrum-averaging routine. Pick peaks on the mean spectrum, rank them by intensity, and select the three highest peaks; for each, extract an m/z window spanning exactly 2 Da (±1 Da around the peak) and plot the corresponding spatial ion-image slices over the tissue section. Cross-check the imaging rendering with the Cardinal package. For protein-centric data, the companion Pbase package provides the central Proteins object — protein sequences from a FASTA file (e.g. UniProt release 2015_02) with peptides from LC-MSMS searches (MSGF+, mzID) mapped as coordinates along the proteins — which the paper's plotting systems visualize as raw and processed MS data. Decision point: the paper covers R's two plotting systems (base and grid/ggplot) — choose based on whether direct manipulation of plot regions or declarative layering better fits the spectral/ion-image layout.

## Related tools

- **R** (Analysis and visualization environment; the paper provides an overview of R's plotting systems for raw and processed MS-based proteomics data)
- **MALDIquant** (Import per-pixel MassSpectrum objects from the MALDI imaging dataset, average spectra across pixels, and pick/rank peaks on the mean spectrum)
- **Cardinal** (Cross-check the rendering of the MALDI imaging ion-image slices)
- **Pbase** (Provides the central Proteins data object linking protein sequences (FASTA) to peptide ranges-of-interest from LC-MSMS/mzID results for visualization) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **mzID** (Reads peptide identification results used to populate Proteins instances)
- **MSGF+** (Search engine that produced the peptide identifications demonstrated in the vignette)

## Examples

```
fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase")
```

## Evaluation signals

- The mean spectrum plus three 2-Da ion-image slices render and together reproduce the layout and content of the paper's Figure 7 (mouse kidney MALDI imaging)
- Each slice window spans exactly 2 Da, centered on one of the three most intense peaks of the mean spectrum (±1 Da around the peak m/z)
- The imported object is a collection of per-pixel MassSpectrum instances covering all imaging pixels before averaging
- Peak ranking on the mean spectrum yields the three highest-intensity peaks before any slice windows are extracted
- The ion-image rendering matches when cross-checked with the Cardinal imaging package

## Limitations

- Pbase is under heavy development and likely to considerably change in the near future, so the data object/API demonstrated in the vignette may shift
- The paper is a short overview of R's plotting systems rather than a benchmarked workflow; no quantitative validation thresholds are given beyond the 2-Da slice-width and top-3-peak selection
- The provided document lacks a discussion section (no changelog found), so caveats from the authors' discussion are not available

## Evidence

- [abstract] Here, we provided a short overview of the application of the R software to the visualization of proteomics data.: "Here, we provided a short overview of the application of the R software to the visualization of proteomics data."
- [abstract] Data visualization plays a key role in high-throughput biology.: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [intro] Pbase objects populated from FASTA and LC-MSMS peptides: "`Proteins` objects are populated by protein sequences stemming from a
fasta file and the peptides typically originate from an LC-MSMS
experiment."
- [other] Mouse kidney MALDI imaging finding (source card): [evidence span withheld — anchor status: not_found]
- [readme] Pbase getting started (README): "Currently, the best way to get started is `?Proteins` and the
[`Pbase-data`](http://bioconductor.org/packages/devel/bioc/vignettes/Pbase/inst/doc/Pbase-data.html)
vignette."
