---
name: lc-ms-raw-data-import
description: Use when a raw MS run (e.g. an mzML/TMT experiment such as PXD000001
  acquired on a Thermo Orbitrap Q Exactive) must be loaded into R at scan level to
  plot retention-time vs m/z heat maps, 3D scan maps, or MS/MS2 spectra.
when_to_use_negative:
- Input is peptide/protein identification results only (mzid files) — use the mzID
  package for PSM identification import rather than mzR raw import
- Data are already processed protein/peptide quantification tables (e.g. MSnSet instances)
  — no raw-scan import is needed
- Task is populating Pbase Proteins objects from FASTA sequences plus LC-MSMS identifications
  rather than visualizing raw scans
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3522
- http://edamontology.org/topic_0091
tools:
- name: mzR
  role: Provides the direct low-level, on-disk access to raw MS data and meta-data
    (openMSfile, peaks, header) used to load the raw run for visualization
- name: Pbase
  role: Bioconductor package context for the vignette workflow; interoperates with
    identification results (mzID or mzR) and MSnExp/MSnSet instances
  repo: github.com/lgatto/Pbase
- name: knitr
  role: Reproducible vignette/figure engine for assembling and rendering the raw-MS
    figure panels
- name: rmarkdown
  role: Document format driving the knitr figure render
- name: BiocStyle
  role: Output style (BiocStyle::html_document) for the rendered vignette/figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:b4813a2f0fc046d05bc04f037323574bc512e60e63e76db9a2abe36831e4b7c0
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3522
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lc-ms-raw-data-import

## Summary

Import raw LC-MS/MS runs into R using mzR's direct low-level, on-disk access to raw MS peak data and meta-data, separating MS1 survey scans from MS2 fragmentation scans so they can be rendered as retention-time heat maps, 3D scan maps, MS/MS2 spectra and isotopic-envelope zooms.

## When to use

Use when a raw MS run (e.g. an mzML/TMT experiment such as PXD000001 acquired on a Thermo Orbitrap Q Exactive) must be loaded into R at scan level to plot retention-time vs m/z heat maps, 3D scan maps, or MS/MS2 spectra.

## When NOT to use

- Input is peptide/protein identification results only (mzid files) — use the mzID package for PSM identification import rather than mzR raw import
- Data are already processed protein/peptide quantification tables (e.g. MSnSet instances) — no raw-scan import is needed
- Task is populating Pbase Proteins objects from FASTA sequences plus LC-MSMS identifications rather than visualizing raw scans

## Inputs

- Raw LC-MS/MS run file (mzML/mzXML-derived, e.g. ProteomeXchange PXD000001 TMT experiment)
- mzR header meta-data (scan index, msLevel, retention time, precursor m/z)
- Instrument metadata (e.g. Thermo Orbitrap Q Exactive acquisition)

## Outputs

- Scan-level peak lists (m/z, intensity per scan)
- Run header meta-data table
- Retention-time vs m/z heat map with color-coded ion intensity
- 3D scan representations (m/z, retention time, intensity) for MS1 and MS2 scans
- MS/MS2 spectrum plots and isotopic-envelope zoom panel, e.g. assembled as figure3_raw_ms_views.png

## How to apply

Open the raw file with mzR::openMSfile, which keeps the data on disk rather than loading everything into memory, then pull scan-level peak lists with mzR::peaks and run-level meta-data with mzR::header. Separate acquisition levels by the msLevel field of the header so MS1 survey scans and MS2 fragmentation scans can be visualized independently. Render the retention-time vs m/z heat map with color-coded ion intensity using R's base/lattice plotting systems, and produce 3D representations (m/z, retention time, intensity) for both MS1 and MS2 scans. For targeted inspection, zoom on a selected peptide signal to show its isotopic envelope. Assemble panels into a figure layout (e.g. Figure 3-style raw-MS views) via knitr/rmarkdown with BiocStyle and export as PNG.

## Related tools

- **mzR** (Provides the direct low-level, on-disk access to raw MS data and meta-data (openMSfile, peaks, header) used to load the raw run for visualization)
- **Pbase** (Bioconductor package context for the vignette workflow; interoperates with identification results (mzID or mzR) and MSnExp/MSnSet instances) — github.com/lgatto/Pbase
- **knitr** (Reproducible vignette/figure engine for assembling and rendering the raw-MS figure panels)
- **rmarkdown** (Document format driving the knitr figure render)
- **BiocStyle** (Output style (BiocStyle::html_document) for the rendered vignette/figure)

## Examples

```
library("mzR"); raw <- openMSfile("PXD000001.mzML"); hd <- header(raw); peaks(raw, scans=which(hd$msLevel==1))
```

## Evaluation signals

- Header meta-data parsed successfully and scans separable by msLevel into MS1 survey scans and MS2 fragmentation scans
- Retention-time vs m/z heat map rendered with color-coded ion intensity across the run's acquisition range
- Both 3D scan views (MS1 and MS2, axes m/z / retention time / intensity) rendered
- Isotopic-envelope zoom clearly shows a selected peptide signal's isotope pattern
- Exported figure3_raw_ms_views.png exists and contains all four panels matching the article's Figure 3 visual content

## Limitations

- mzR access is low-level and on-disk: the analyst must manually separate MS levels and build all plots; no ready-made raw-figure functions are provided
- Pbase was under heavy development and likely to considerably change; Pbase interoperates with mzR results but the package was still evolving at the time of the article
- The demonstrated run is a single Thermo Orbitrap Q Exactive example (PXD000001); rendering quality depends on scan density and isotope intensity of the selected peptide signal

## Evidence

- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [intro] acquired on a Thermo Orbitrap Q Exactive instrument.: "acquired on a Thermo Orbitrap Q Exactive instrument."
- [readme] protein identification results (`mzID` or (devel) `mzR` packages) and possibly also `MSnExp` and `MSnSet` instances.: [evidence span withheld — anchor status: external_doc]
