---
name: spectral-smoothing-and-baseline-correction
description: Use when a raw MALDI-TOF (or other 2D MS1-level) mass spectrum loaded
  in R must be cleaned for peak picking, and each preprocessing stage (transformed,
  smoothed, baseline-corrected, peak-picked) needs to be visually inspectable.
when_to_use_negative:
- The input is already a peak table or processed feature matrix rather than a raw
  MS1 mass spectrum
- The task concerns protein/peptide identification data (e.g. Pbase Proteins objects
  from mzID/FASTA) rather than raw spectral intensity data
- Quantitative comparison of preprocessing models across different devices/datasets
  is required (e.g. MALDI vs ESI), which the source material warns against
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MALDIquant
  role: Provides the single-spectrum preprocessing chain (intensity transformation,
    smoothing, baseline removal, peak detection) and traditional-graphics plotting
    for step-by-step inspection
- name: R
  role: Execution environment in which the MALDIquant preprocessing and plotting chain
    is run
- name: knitr
  role: Vignette/reporting engine used to document and render the visualization workflow
- name: rmarkdown
  role: Vignette output format (%\VignetteEngine{knitr::rmarkdown}) for the documented
    workflow
- name: BiocStyle
  role: Bioconductor document style (BiocStyle::html_document) used for the rendered
    vignette
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:4d60a46ff682034b198790c094a85f47e1e93cf8422e39fecb37aabb59d2262b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MALDIquant
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spectral-smoothing-and-baseline-correction

## Summary

A MALDIquant-based R chain for single-spectrum MALDI-TOF preprocessing: intensity transformation for variance stabilization, smoothing, chemical baseline removal, and peak detection, with each intermediate stage rendered for detailed inspection (as in Fig. 5 of the paper).

## When to use

Use when a raw MALDI-TOF (or other 2D MS1-level) mass spectrum loaded in R must be cleaned for peak picking, and each preprocessing stage (transformed, smoothed, baseline-corrected, peak-picked) needs to be visually inspectable.

## When NOT to use

- The input is already a peak table or processed feature matrix rather than a raw MS1 mass spectrum
- The task concerns protein/peptide identification data (e.g. Pbase Proteins objects from mzID/FASTA) rather than raw spectral intensity data
- Quantitative comparison of preprocessing models across different devices/datasets is required (e.g. MALDI vs ESI), which the source material warns against

## Inputs

- Single MALDI-TOF mass spectrum (mass/intensity pairs) as a MALDIquant MassSpectrum object
- Example MALDI-TOF mass spectrum shipped as demonstration data with MALDIquant

## Outputs

- Transformed, smoothed, and baseline-corrected MassSpectrum objects (one per preprocessing stage)
- List of single peaks from peak detection on the baseline-corrected spectrum
- Multi-panel figure of the preprocessing stages saved as figure5_maldiquant_preprocessing_panels.png

## How to apply

Load a demonstration or user MALDI-TOF mass spectrum into MALDIquant as a MassSpectrum object. First apply the intensity transformation to stabilize variance across the mass range, then smooth the transformed spectrum and remove the chemical baseline so the signal is flattened onto zero; finally run peak detection on the baseline-corrected spectrum to extract the list of single peaks. Use MALDIquant's plotting functions, built on the traditional R graphics infrastructure, to render the successive stages (raw, transformed, smoothed, baseline-corrected, peak-picked) as a multi-panel figure and save it as figure5_maldiquant_preprocessing_panels.png. The rationale is that the traditional-graphics plotting is fast and flexible, allowing inspection of every single preprocessing step rather than only the final result.

## Related tools

- **MALDIquant** (Provides the single-spectrum preprocessing chain (intensity transformation, smoothing, baseline removal, peak detection) and traditional-graphics plotting for step-by-step inspection)
- **R** (Execution environment in which the MALDIquant preprocessing and plotting chain is run)
- **knitr** (Vignette/reporting engine used to document and render the visualization workflow)
- **rmarkdown** (Vignette output format (%\VignetteEngine{knitr::rmarkdown}) for the documented workflow)
- **BiocStyle** (Bioconductor document style (BiocStyle::html_document) used for the rendered vignette)

## Examples

```
library(MALDIquant); s <- transformIntensity(spec, method="sqrt"); s <- smoothIntensity(s, method="SavitzkyGolay", halfWindowSize=10); s <- removeBaseline(s); peaks <- detectPeaks(s)
```

## Evaluation signals

- After baseline correction, the spectrum signal is flattened onto zero (no residual chemical baseline visible in the plotted stage)
- Peak detection on the corrected spectrum yields a non-empty list of single peaks consistent with visible spectral features
- The rendered multi-panel figure contains all five successive stages: raw spectrum, transformed, smoothed, baseline-corrected, and peak-picked
- The figure is saved under the expected filename figure5_maldiquant_preprocessing_panels.png and is reproducible from the MALDIquant demonstration spectrum
- Each intermediate stage can be individually plotted/inspected via MALDIquant's plotting functions, confirming stepwise traceability

## Limitations

- Demonstrated for a single MALDI-TOF spectrum (single-spectrum inspection), not a batched multi-spectrum processing pipeline
- The paper provides a short overview; detailed parameter tuning for the transformation, smoothing, and baseline steps is not given in the provided context
- The source article's discussion section text was not available, limiting caveats beyond those stated in abstract/intro

## Evidence

- [other] MALDIquant's plotting functions, based on the traditional graphics infrastructure, are fast and flexible and allow the user to investigate each single preprocessing step in detail, as illustrated in Fig. 5.: [evidence span withheld — anchor status: not_found]
- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [other] Smooth the transformed spectrum and remove the chemical baseline so that the signal is flattened onto zero.: [evidence span withheld — anchor status: not_found]
- [other] Render the successive preprocessing stages (raw spectrum, transformed, smoothed, baseline-corrected, peak-picked) as the multi-panel figure: [evidence span withheld — anchor status: not_found]
- [methods] %\VignetteEngine{knitr::rmarkdown}: "%\VignetteEngine{knitr::rmarkdown}"
