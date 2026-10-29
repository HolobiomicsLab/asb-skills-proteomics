---
name: multi-panel-spectra-figure-rendering
description: Use when you need to visually inspect each step of a MALDI-TOF or 2D
  MS1-level spectrum preprocessing chain, e.g. to verify variance stabilization, smoothing,
  baseline removal, or peak detection on a single mass spectrum.
when_to_use_negative:
- Input is already a processed feature/peak table rather than a raw mass spectrum
  — no preprocessing stages exist to render
- The goal is manipulation of protein/peptide identification data (e.g. Pbase Proteins
  objects from FASTA + mzID) rather than raw MS1-level spectral inspection
- MS2-level or LC-MSMS tandem spectra requiring fragmentation-oriented visualization
  rather than single-spectrum MALDI-TOF preprocessing panels
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MALDIquant
  role: Provides the spectrum preprocessing functions (transformIntensity, smoothIntensity,
    removeBaseline, detectPeaks) and the traditional-graphics plotting functions used
    to render each preprocessing stage as a figure panel
- name: R
  role: Runtime environment in which the demonstration MALDI-TOF spectrum is loaded,
    the preprocessing chain is executed, and the multi-panel figure is produced and
    saved as PNG
- name: knitr
  role: Vignette engine used to document and reproduce the R-based visualization workflow
    alongside the rendered figure
- name: rmarkdown
  role: Vignette engine (knitr::rmarkdown) used to render the reproducible R workflow
    document containing the figure
- name: BiocStyle
  role: Document styling (BiocStyle::html_document) applied to the reproducible workflow
    output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:9147148b7185e3e149ebb5d5dba4251ee7d28b4d9e7be269bc5919902c696c0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MALDIquant
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-panel-spectra-figure-rendering

## Summary

Render the successive MALDIquant single-spectrum preprocessing stages (raw, transformed, smoothed, baseline-corrected, peak-picked) as a multi-panel figure reproducing Fig. 5, so that each preprocessing step can be visually inspected and validated.

## When to use

Use when you need to visually inspect each step of a MALDI-TOF or 2D MS1-level spectrum preprocessing chain, e.g. to verify variance stabilization, smoothing, baseline removal, or peak detection on a single mass spectrum.

## When NOT to use

- Input is already a processed feature/peak table rather than a raw mass spectrum — no preprocessing stages exist to render
- The goal is manipulation of protein/peptide identification data (e.g. Pbase Proteins objects from FASTA + mzID) rather than raw MS1-level spectral inspection
- MS2-level or LC-MSMS tandem spectra requiring fragmentation-oriented visualization rather than single-spectrum MALDI-TOF preprocessing panels

## Inputs

- Single MALDI-TOF mass spectrum (mass/intensity pairs), e.g. example data shipped with MALDIquant in an R session

## Outputs

- Multi-panel PNG figure (figure5_maldiquant_preprocessing_panels.png) showing raw, transformed, smoothed, baseline-corrected, and peak-picked stages of the spectrum
- List of detected single peaks from the baseline-corrected spectrum

## How to apply

Load a demonstration MALDI-TOF mass spectrum shipped as MALDIquant example data in R, then apply the preprocessing chain in order: transformIntensity to stabilize variance across the mass range, smoothIntensity to smooth the transformed spectrum, removeBaseline to remove the chemical baseline and flatten the signal onto zero, and detectPeaks to extract the peak list from the baseline-corrected spectrum. Render each stage (raw, transformed, smoothed, baseline-corrected, peak-picked) into a separate panel using MALDIquant's plotting functions, which are based on the traditional graphics infrastructure and are fast and flexible enough to investigate each single preprocessing step in detail. Arrange the panels as the multi-panel figure and save it as figure5_maldiquant_preprocessing_panels.png. Correct application is judged by visually confirming that the transformed spectrum is variance-stabilized, the baseline-corrected spectrum is flattened onto zero, and detected peaks align with visible intensity maxima.

## Related tools

- **MALDIquant** (Provides the spectrum preprocessing functions (transformIntensity, smoothIntensity, removeBaseline, detectPeaks) and the traditional-graphics plotting functions used to render each preprocessing stage as a figure panel)
- **R** (Runtime environment in which the demonstration MALDI-TOF spectrum is loaded, the preprocessing chain is executed, and the multi-panel figure is produced and saved as PNG)
- **knitr** (Vignette engine used to document and reproduce the R-based visualization workflow alongside the rendered figure)
- **rmarkdown** (Vignette engine (knitr::rmarkdown) used to render the reproducible R workflow document containing the figure)
- **BiocStyle** (Document styling (BiocStyle::html_document) applied to the reproducible workflow output)

## Examples

```
library(MALDIquant); s <- transformIntensity(fiedler2009subset[[1]], method="sqrt"); s <- smoothIntensity(s); s <- removeBaseline(s); peaks <- detectPeaks(s)
```

## Evaluation signals

- The output PNG figure5_maldiquant_preprocessing_panels.png exists and contains five panels corresponding to raw, transformed, smoothed, baseline-corrected, and peak-picked stages in order
- The baseline-corrected panel shows the signal flattened onto zero (chemical baseline removed)
- Detected peaks in the final panel coincide with visible intensity maxima of the baseline-corrected spectrum
- The transformed panel shows variance stabilized across the full mass range compared to the raw spectrum
- Each panel is rendered via MALDIquant's own plotting functions rather than generic plotting, preserving per-step inspectability

## Limitations

- MALDIquant's plotting functions are built on the traditional graphics infrastructure, so users preferring grid/ggplot2-based graphics must adapt the rendering
- The panel chain targets single MALDI-TOF / MS1-level spectra; it does not cover MS2 or LC-MSMS identification data
- The article's extracted entities contain no discussion-section text, and the accompanying repository README documents the Pbase package, not MALDIquant, so repo-level guidance for this skill is limited

## Evidence

- [other] MALDIquant plotting quote: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract plotting-systems quote: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [abstract] Visualization-as-exploration quote: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [methods] Methods vignette-engine quote: "%\VignetteEngine{knitr::rmarkdown}"
- [full_text] Workflow steps quote: [evidence span withheld — anchor status: not_found]
