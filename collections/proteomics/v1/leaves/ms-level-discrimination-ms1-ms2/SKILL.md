---
name: ms-level-discrimination-ms1-ms2
description: Use when a raw MS run (e.g. mzML/mzXML from ProteomeXchange) must be
  split into MS1 survey vs MS2 fragmentation scans before rendering retention-time
  heat maps, 3D m/z-RT-intensity maps, or isotopic-envelope zooms.
when_to_use_negative:
- Input is identification results (e.g. an .mzid file) rather than raw scan-level
  data — mzR's low-level raw access is not applicable
- Data are already processed into feature/quantification tables (e.g. MSnSet instances);
  the skill targets raw-run exploration, not processed-data plotting
- The task is protein-sequence or peptide-to-protein coordinate mapping (Pbase Proteins
  ranges), which does not require MS1/MS2 scan discrimination
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: mzR
  role: Provides direct low-level, on-disk access to raw MS data and meta-data used
    to extract and level-split scans for the Figure 3 visualizations
- name: Pbase
  role: Bioconductor package context for proteomics data manipulation/visualization
    in the article; interoperates with mzID/mzR identification and MSnExp/MSnSet objects
  repo: github.com/lgatto/Pbase
- name: knitr
  role: Vignette/figure engine used to assemble and render the multi-panel raw MS
    figure
- name: rmarkdown
  role: Vignette engine (%\VignetteEngine{knitr::rmarkdown}) for reproducible figure
    generation
- name: BiocStyle
  role: Output style (BiocStyle::html_document) for the rendered vignette/figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:a1a7a47b8a34b018982583ae21ba298ff88474aab92b2d6b8f0a1fbfc558c4ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - Pbase
  - R
  - knitr
  - mzR
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ms-level-discrimination-ms1-ms2

## Summary

Separates scan-level data from a raw LC-MS/MS run into MS1 survey scans and MS2 fragmentation scans by acquisition level, using mzR's low-level on-disk access, so that raw-run visualizations (retention-time heat map, 3D scan maps, isotopic-envelope zoom) can be rendered level-by-level in R.

## When to use

Use when a raw MS run (e.g. mzML/mzXML from ProteomeXchange) must be split into MS1 survey vs MS2 fragmentation scans before rendering retention-time heat maps, 3D m/z-RT-intensity maps, or isotopic-envelope zooms.

## When NOT to use

- Input is identification results (e.g. an .mzid file) rather than raw scan-level data — mzR's low-level raw access is not applicable
- Data are already processed into feature/quantification tables (e.g. MSnSet instances); the skill targets raw-run exploration, not processed-data plotting
- The task is protein-sequence or peptide-to-protein coordinate mapping (Pbase Proteins ranges), which does not require MS1/MS2 scan discrimination

## Inputs

- Raw MS run file from a ProteomeXchange deposit (e.g. PXD000001 TMT experiment)
- Scan-level peak lists and scan meta-data accessible via mzR (retention time, m/z, intensity, acquisition level)
- R session with knitr/rmarkdown/BiocStyle for figure assembly

## Outputs

- Retention-time vs m/z heat map with color-coded ion intensity
- 3D representations (m/z, retention time, intensity) of MS1 and MS2 scans
- Zoomed isotopic-envelope view of a selected peptide signal
- Assembled multi-panel figure (e.g. figure3_raw_ms_views.png)

## How to apply

Load the raw run into R with mzR, which provides direct low-level, on-disk access to raw MS data and meta-data (for the article's example, the PXD000001 TMT experiment acquired on a Thermo Orbitrap Q Exactive instrument from a 10 fmol Peptide Retention Time Calibration Mixture). Extract scan-level peak data and scan headers, then partition scans by acquisition level so MS1 survey scans and MS2 fragmentation scans are handled as separate sets. Use the MS1 set for the retention-time vs m/z heat map with color-coded ion intensity and the 3D (m/z, retention time, intensity) representations; use the MS2 set for fragmentation-scan views. Produce a zoomed view of a selected peptide signal highlighting its isotopic envelope. Assemble the panels (e.g. via knitr/rmarkdown with BiocStyle, as in the article's vignette engine) and export as figure3_raw_ms_views.png.

## Related tools

- **mzR** (Provides direct low-level, on-disk access to raw MS data and meta-data used to extract and level-split scans for the Figure 3 visualizations)
- **Pbase** (Bioconductor package context for proteomics data manipulation/visualization in the article; interoperates with mzID/mzR identification and MSnExp/MSnSet objects) — github.com/lgatto/Pbase
- **knitr** (Vignette/figure engine used to assemble and render the multi-panel raw MS figure)
- **rmarkdown** (Vignette engine (%\VignetteEngine{knitr::rmarkdown}) for reproducible figure generation)
- **BiocStyle** (Output style (BiocStyle::html_document) for the rendered vignette/figure)

## Examples

```
devtools::install_github("ComputationalProteomicsUnit/Pbase")
```

## Evaluation signals

- Every scan in the run is assigned to exactly one acquisition level (MS1 survey or MS2 fragmentation), with no unclassified scans
- The heat map axes are retention time (x) vs m/z (y) with ion intensity as the color scale
- Both 3D views (MS1 and MS2 scans) are rendered in m/z, retention time, intensity space
- The isotopic-envelope zoom shows an equally spaced peak cluster for the selected peptide signal
- The exported figure3_raw_ms_views.png exists and its four panels match the visual content of Figure 3 in the article

## Limitations

- Pbase is 'under heavy development and is likely to considerably change in the near future', so surrounding tooling may shift
- The README notes raw identification interoperability via the '(devel)' mzR package, i.e. low-level raw access depends on a development package
- Demonstrated on a single example run (10 fmol Peptide Retention Time Calibration Mixture on a Thermo Orbitrap Q Exactive); thresholds or scan distributions will differ for other instruments/datasets
- No discussion-section text was available in the provided document, so caveats beyond those stated are unverified

## Evidence

- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [intro] acquired on a Thermo Orbitrap Q Exactive instrument.: "acquired on a Thermo Orbitrap Q Exactive instrument."
- [readme] protein identification results (`mzID` or (devel) `mzR` packages) and possibly also `MSnExp` and `MSnSet` instances.: [evidence span withheld — anchor status: external_doc]
- [readme] Currently, the best way to get started is `?Proteins` and the [`Pbase-data`](http://bioconductor.org/packages/devel/bioc/vignettes/Pbase/inst/doc/Pbase-data.html) vignette.: [evidence span withheld — anchor status: external_doc]
