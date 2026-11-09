---
name: retention-time-mz-heatmap-rendering
description: 'Use when you hold an on-disk raw MS file (e.g. the TMT PXD000001 run
  from a Thermo Orbitrap Q Exactive) and need exploratory visual inspection: ion-intensity
  heat maps over retention time and m/z, 3D scan maps, or a zoom on a peptide''s isotopic
  envelope.'
when_to_use_negative:
- The data is already identified and quantified (e.g. a Proteins/MSnSet feature table)
  — use protein/peptide-level plotting on Pbase Proteins instances instead of raw-scan
  rendering
- You need processed-data plots (identifications, protein coverage, peptide mapping)
  rather than exploratory raw-run views
- The instrument vendor format is not directly readable by mzR and you have not yet
  converted it (e.g. to mzML)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzR
  role: Provides direct low-level, on-disk access to the raw MS data and meta-data
    that feed the heat map, 3D scan maps, and isotopic-envelope zoom
- name: knitr
  role: Vignette/figure engine used to assemble and render the figure panels reproducibly
- name: rmarkdown
  role: Reproducible report driver (`knitr::rmarkdown` vignette engine) for the figure
    build
- name: BiocStyle
  role: Output document style (`BiocStyle::html_document`) for the rendering workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:d9a2d599b7dbb4ceb3d7da1e196827d1d203aeb9e577c547da9a18b3d5153145
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# retention-time-mz-heatmap-rendering

## Summary

Render exploratory views of a raw LC-MS/MS run — retention-time vs m/z intensity heat maps, 3D (m/z, RT, intensity) scan maps for MS1 and MS2 scans, and isotopic-envelope zooms — using mzR's low-level on-disk access to raw MS data and R's base/lattice plotting.

## When to use

Use when you hold an on-disk raw MS file (e.g. the TMT PXD000001 run from a Thermo Orbitrap Q Exactive) and need exploratory visual inspection: ion-intensity heat maps over retention time and m/z, 3D scan maps, or a zoom on a peptide's isotopic envelope.

## When NOT to use

- The data is already identified and quantified (e.g. a Proteins/MSnSet feature table) — use protein/peptide-level plotting on Pbase Proteins instances instead of raw-scan rendering
- You need processed-data plots (identifications, protein coverage, peptide mapping) rather than exploratory raw-run views
- The instrument vendor format is not directly readable by mzR and you have not yet converted it (e.g. to mzML)

## Inputs

- Raw MS file for a single LC-MS/MS run on disk (e.g. the PXD000001 TMT experiment deposit from ProteomeXchange), readable by mzR
- Scan-level acquisition meta-data (acquisition level MS1 vs MS2, retention time) from the mzR scan headers

## Outputs

- figure3_raw_ms_views.png containing the retention-time vs m/z intensity heat map, 3D MS1 and MS2 scan views, and an isotopic-envelope zoom panel
- Scan-level peak tables split by MS level

## How to apply

Open the raw run with mzR (which provides direct low-level, on-disk access to raw MS data and meta-data), extract scan-level peak lists, and separate MS1 survey scans from MS2 fragmentation scans by acquisition level using the scan-header meta-data. Bin the peaks onto a retention-time × m/z grid, map ion intensity to color, and render the heat map with R's base/lattice plotting systems. Render 3D representations (m/z, retention time, intensity) of both the MS1 and MS2 scans, then produce a zoomed view highlighting the isotopic envelope of a selected peptide signal (~1/z charge-state spacing). Assemble the panels into a figure layout via knitr/rmarkdown with BiocStyle and export as a single PNG.

## Related tools

- **mzR** (Provides direct low-level, on-disk access to the raw MS data and meta-data that feed the heat map, 3D scan maps, and isotopic-envelope zoom)
- **knitr** (Vignette/figure engine used to assemble and render the figure panels reproducibly)
- **rmarkdown** (Reproducible report driver (`knitr::rmarkdown` vignette engine) for the figure build)
- **BiocStyle** (Output document style (`BiocStyle::html_document`) for the rendering workflow)

## Examples

```
library(mzR); fh <- mzR::openMSfile("PXD000001.mzML"); ms <- mzR::peaks(fh); hdr <- mzR::header(fh)
```

## Evaluation signals

- All four panels are rendered: the RT×m/z intensity heat map, the 3D MS1 view, the 3D MS2 view, and the isotopic-envelope zoom are all present in the exported PNG
- The exported figure3_raw_ms_views.png exists and its visual content matches Figure 3 of the article (heat map, both 3D scan views, envelope zoom clearly visible)
- MS1 survey scans and MS2 fragmentation scans are correctly separated by acquisition level before plotting
- Heat map color intensity is scaled to measured ion intensity, and the envelope zoom shows ~1/z m/z spacing between isotope peaks of the selected peptide signal

## Limitations

- mzR requires the raw run accessible on disk; the visualization targets a single raw run and does not operate on identification results
- Pbase and its vignette workflows were 'under heavy development and is likely to considerably change in the near future', so panel-reproduction code may need adjustment
- The article's demonstration data is a 10 fmol Peptide Retention Time Calibration Mixture run on a Thermo Orbitrap Q Exactive; scale/intensity behavior may differ for other instruments

## Evidence

- [abstract] R plotting systems for raw MS visualization: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [abstract] Visualization for data exploration: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [intro] Instrument context for the demonstrated run: "acquired on a Thermo Orbitrap Q Exactive instrument."
- [readme] mzR interoperability with raw data packages: [evidence span withheld — anchor status: external_doc]
