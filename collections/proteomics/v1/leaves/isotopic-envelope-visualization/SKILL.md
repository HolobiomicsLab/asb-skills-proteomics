---
name: isotopic-envelope-visualization
description: 'Use when you have a raw MS run (e.g. PXD000001, TMT experiment from
  a Thermo Orbitrap Q Exactive) and need scan-level visual exploration: heat maps,
  3D scan maps, or isotopic-envelope zooms, in R.'
when_to_use_negative:
- Input is already processed, e.g. identification results in mzid files or a feature/quantitation
  table — use Pbase/MSnSet-level visualization instead of raw scan rendering
- The goal is protein/peptide-level exploration of sequences and mapped peptide ranges,
  which is Pbase's Proteins-object workflow rather than raw-run visualization
- No R/Bioconductor environment is available and the data must be explored interactively
  in vendor raw-file software
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mzR
  role: Provides direct low-level, on-disk access to raw MS data and meta-data used
    to extract scan-level peaks and msLevel annotations for the Figure 3 visualizations
- name: Pbase
  role: Article's companion package whose example raw data (Thermo_Hela_PRTC files)
    and ecosystem the raw-run demonstrations sit within; also interoperates with mzR
    for identification results
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: knitr
  role: Vignette engine used to render the visualization workflow and export figure
    panels reproducibly
- name: rmarkdown
  role: Document engine (knitr::rmarkdown) for assembling the figure workflow
- name: BiocStyle
  role: Output styling (BiocStyle::html_document) for the rendered visualization document
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:345294d497c806ac863f9b392c4f0029a106da582130a07f008c322b5a9d381f
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# isotopic-envelope-visualization

## Summary

Renders raw MS visualizations of an LC-MS/MS run — a retention-time vs m/z ion-intensity heat map, 3D m/z × retention-time × intensity maps of MS1 survey and MS2 fragmentation scans, and a zoomed view of a peptide's isotopic envelope — directly from on-disk raw data using mzR, reproducing Figure 3 of Gatto et al. 2015.

## When to use

Use when you have a raw MS run (e.g. PXD000001, TMT experiment from a Thermo Orbitrap Q Exactive) and need scan-level visual exploration: heat maps, 3D scan maps, or isotopic-envelope zooms, in R.

## When NOT to use

- Input is already processed, e.g. identification results in mzid files or a feature/quantitation table — use Pbase/MSnSet-level visualization instead of raw scan rendering
- The goal is protein/peptide-level exploration of sequences and mapped peptide ranges, which is Pbase's Proteins-object workflow rather than raw-run visualization
- No R/Bioconductor environment is available and the data must be explored interactively in vendor raw-file software

## Inputs

- Raw MS run file (mzML/mzXML) from a ProteomeXchange deposit, e.g. PXD000001 TMT experiment acquired on a Thermo Orbitrap Q Exactive
- Peptide Retention Time Calibration Mixture sample data
- Scan header and peak-list data accessible via mzR's on-disk interface

## Outputs

- Retention-time vs m/z heat map with color-coded ion intensity
- 3D maps (m/z, retention time, intensity) of MS1 and MS2 scans
- Zoomed isotopic-envelope view of a selected peptide signal
- figure3_raw_ms_views.png assembled via knitr/rmarkdown with BiocStyle

## How to apply

Load the raw ProteomeXchange run (e.g. the 10 fmol Peptide Retention Time Calibration Mixture PXD000001 file) with the mzR package, which provides direct low-level, on-disk access to raw MS data and meta-data. Extract scan-level peak lists and header meta-data, then separate MS1 survey scans from MS2 fragmentation scans by acquisition level (msLevel). Render the retention-time vs m/z heat map with color-coded ion intensity using R's base/lattice plotting systems, and build 3D representations (m/z, retention time, intensity) for both MS1 and MS2 scans. For the isotopic-envelope zoom, select a high-intensity peptide signal and expand the m/z axis around it so the co-eluting isotope peaks are resolved. Assemble the panels via knitr/rmarkdown with BiocStyle and export as a single PNG (e.g. figure3_raw_ms_views.png).

## Related tools

- **mzR** (Provides direct low-level, on-disk access to raw MS data and meta-data used to extract scan-level peaks and msLevel annotations for the Figure 3 visualizations)
- **Pbase** (Article's companion package whose example raw data (Thermo_Hela_PRTC files) and ecosystem the raw-run demonstrations sit within; also interoperates with mzR for identification results) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **knitr** (Vignette engine used to render the visualization workflow and export figure panels reproducibly)
- **rmarkdown** (Document engine (knitr::rmarkdown) for assembling the figure workflow)
- **BiocStyle** (Output styling (BiocStyle::html_document) for the rendered visualization document)

## Examples

```
library(mzR); fh <- openMSfile("PXD000001.mzML"); hd <- header(fh); ms1 <- hd[hd$msLevel == 1, ]; p <- peaks(fh, which.max(hd$basePeakIntensity[hd$msLevel == 1])); plot(p[,1], p[,2], type="h", xlim=c(p[1,1]-3, p[1,1]+5))
```

## Evaluation signals

- All four Figure 3 panels are rendered: the RT-vs-m/z heat map, the MS1 and MS2 3D scan maps, and the isotopic-envelope zoom are clearly present
- The exported PNG (figure3_raw_ms_views.png) exists and matches the visual content of Figure 3 in the article
- MS1 survey scans and MS2 fragmentation scans are correctly separated by acquisition level before plotting the 3D maps
- The zoomed panel resolves the isotopic envelope — multiple co-eluting isotope peaks of a single peptide — rather than a single unexpanded peak
- Peak data is read via mzR directly from the on-disk ProteomeXchange file (no intermediate conversion or pre-extracted tables)

## Limitations

- mzR provides low-level access only; all plotting and scan-type separation must be hand-coded with base/lattice graphics
- Pbase was described as under heavy development and likely to change considerably in the near future, so companion interfaces may shift
- Figure 3 demonstrates a single example MS run (PXD000001, Orbitrap Q Exactive, PRTC mixture); visual quality depends on scan density and dynamic range of the instrument acquisition

## Evidence

- [results] mzR on-disk access finding: [evidence span withheld — anchor status: not_found]
- [readme] interoperability with mzR: [evidence span withheld — anchor status: external_doc]
- [intro] instrument and data provenance: "The original data used below is a 10 fmol [Peptide Retention Time Calibration Mixture](http://www.piercenet.com/product/peptide-retention-time-calibration-mixture)"
- [methods] rendering pipeline: "%\VignetteEngine{knitr::rmarkdown}"
- [abstract] article scope: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
