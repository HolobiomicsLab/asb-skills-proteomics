---
name: 3d-scan-intensity-visualization
description: Use when the input is a raw MS run (e.g. a Thermo Orbitrap Q Exactive
  acquisition from ProteomeXchange) and the goal is exploratory plotting of scan-level
  peak data — heat maps, 3D scan maps, or isotopic-envelope zooms — rather than identification
  or quantification.
when_to_use_negative:
- The input is already an identification or feature table (e.g. mzID PSM results or
  an MSnSet quantitation matrix) rather than raw scan-level peak data
- The goal is protein/peptide identification or annotation mapping — for that, the
  paper's workflow uses mzID/MSGF+ results and Pbase Proteins objects populated from
  a fasta file, not raw-run plotting
- Only processed chromatographic peaks or reporter-ion quantitation values are available
  and the raw on-disk file is not accessible via mzR
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mzR
  role: Provides the direct low-level, on-disk access to raw MS data and meta-data
    used to extract scan-level peaks for the Figure 3 raw-run visualizations
- name: R
  role: Analysis and plotting environment; base/lattice plotting systems render the
    heat map and 3D scan views
- name: knitr
  role: Vignette/report engine used to assemble and render the figure-generating document
- name: rmarkdown
  role: Vignette format (%\VignetteEngine{knitr::rmarkdown}) used to build the rendered
    output containing the figure
- name: BiocStyle
  role: Document style (BiocStyle::html_document) applied to the rendered figure report
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:ff4349a84860b5ebfe2bac3d7b995662b1c9d8e4f5a53822c31df6c5341f430c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - R
  - knitr
  - mzR
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# 3D scan intensity visualization

## Summary

A skill for rendering raw mass-spectrometry run views — a retention-time vs m/z heat map with color-coded ion intensity, 3D (m/z, retention time, intensity) representations of MS1 survey and MS2 fragmentation scans, and a zoomed isotopic-envelope view — as demonstrated for the TMT PXD000001 run in Figure 3 of Gatto et al. (2015), using mzR's low-level on-disk access.

## When to use

Use when the input is a raw MS run (e.g. a Thermo Orbitrap Q Exactive acquisition from ProteomeXchange) and the goal is exploratory plotting of scan-level peak data — heat maps, 3D scan maps, or isotopic-envelope zooms — rather than identification or quantification.

## When NOT to use

- The input is already an identification or feature table (e.g. mzID PSM results or an MSnSet quantitation matrix) rather than raw scan-level peak data
- The goal is protein/peptide identification or annotation mapping — for that, the paper's workflow uses mzID/MSGF+ results and Pbase Proteins objects populated from a fasta file, not raw-run plotting
- Only processed chromatographic peaks or reporter-ion quantitation values are available and the raw on-disk file is not accessible via mzR

## Inputs

- Raw MS run file from a ProteomeXchange deposit (e.g. PXD000001, a TMT experiment acquired on a Thermo Orbitrap Q Exactive)
- Scan-level peak lists and acquisition meta-data (msLevel, retention time, m/z, intensity) extracted via mzR

## Outputs

- Retention-time vs m/z heat map with color-coded ion intensity
- 3D (m/z, retention time, intensity) representations of MS1 and MS2 scans
- Zoomed panel highlighting the isotopic envelope of a selected peptide signal
- Assembled multi-panel figure, e.g. figure3_raw_ms_views.png, rendered via knitr/rmarkdown with BiocStyle

## How to apply

Load the raw run (e.g. the PXD000001 TMT experiment) with the mzR package, which provides direct low-level, on-disk access to raw MS data and meta-data. Use the header/peak-list accessors to extract scan-level peaks and separate MS1 survey scans from MS2 fragmentation scans by their acquisition (msLevel) metadata. Render the retention-time vs m/z heat map with color-coded ion intensity, then the 3D (m/z, retention time, intensity) representations of MS1 and MS2 scans, using R's base/lattice plotting systems. Produce a zoomed panel highlighting the isotopic envelope of a selected peptide signal, and assemble all four panels into the Figure 3 layout (saved e.g. as figure3_raw_ms_views.png) via knitr/rmarkdown with BiocStyle.

## Related tools

- **mzR** (Provides the direct low-level, on-disk access to raw MS data and meta-data used to extract scan-level peaks for the Figure 3 raw-run visualizations)
- **R** (Analysis and plotting environment; base/lattice plotting systems render the heat map and 3D scan views)
- **knitr** (Vignette/report engine used to assemble and render the figure-generating document)
- **rmarkdown** (Vignette format (%\VignetteEngine{knitr::rmarkdown}) used to build the rendered output containing the figure)
- **BiocStyle** (Document style (BiocStyle::html_document) applied to the rendered figure report)

## Examples

```
library(mzR); fh <- openMSfile("PXD000001.mzML"); h <- header(fh); p <- peaks(fh, which(h$msLevel == 1))
```

## Evaluation signals

- All four panels are rendered: the retention-time vs m/z heat map, both 3D scan views (MS1 and MS2), and the isotopic-envelope zoom
- The exported PNG (figure3_raw_ms_views.png) exists on disk and the rendered vignette completes under the knitr::rmarkdown engine
- MS1 and MS2 scans are correctly separated by acquisition level, consistent with the run's header msLevel meta-data
- The heat map's color scale maps ion intensity and the m/z / retention-time axes correspond to the raw scan data, matching the visual content of Figure 3 in the article

## Limitations

- Pbase is 'under heavy development and is likely to considerably change in the near future' (README), so downstream interfaces may shift
- The demo relies on a specific example acquisition (PXD000001, Thermo Orbitrap Q Exactive); raw vendor formats other than those readable via mzR's on-disk backends may need conversion
- 3D renderings are exploratory and memory-bound by scan-level peak extraction from large raw files

## Evidence

- [abstract] Plotting systems overview: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [results] mzR low-level access (source card finding): [evidence span withheld — anchor status: not_found]
- [intro] Example raw data source: "The original data used below is a 10 fmol [Peptide Retention Time Calibration Mixture]"
- [readme] mzR availability via README interoperability: [evidence span withheld — anchor status: external_doc]
- [methods] Report rendering setup: "%\VignetteEngine{knitr::rmarkdown}"
