---
name: maldi-tof-spectrum-preprocessing
description: Use when a single MALDI-TOF (2D MS1-level) spectrum must be transformed,
  smoothed, baseline-corrected and peak-picked, and each preprocessing step needs
  to be visually inspected before downstream analysis.
when_to_use_negative:
- Input is already a peak list or feature table rather than a raw MALDI-TOF mass spectrum
  — the transformation/smoothing/baseline-correction chain would be meaningless
- Data comes from LC-MSMS peptide identification workflows (e.g. Pbase/mzID Proteins
  objects or mzID files) rather than 2D MS1 MALDI-TOF spectra
- You need to compare peptide detectability models across devices/datasets — the Pbase
  README warns 'you can't compare different devices/datasets (e.g. MALDI vs ESI)'
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MALDIquant
  role: R framework for processing and analysis of MALDI-TOF and other 2D MS1-level
    data; provides the intensity transformation, smoothing, baseline-correction, peak-detection
    and traditional-graphics plotting functions used in the Fig. 5 chain
- name: R
  role: Runtime environment in which the MALDIquant preprocessing and multi-panel
    plotting workflow is executed
- name: knitr
  role: Vignette engine used to render the R/Bioconductor visualization documentation
    that illustrates the workflow
- name: rmarkdown
  role: Vignette engine (knitr::rmarkdown) used alongside BiocStyle for the reproducible
    R document showing the plotting workflow
- name: BiocStyle
  role: Output styling (BiocStyle::html_document) for the reproducible vignette presenting
    the workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:afa0cda44a8fca034b3b8b0673f999b8ea3ecced1e608685902f6e52b34f98aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MALDIquant
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# maldi-tof-spectrum-preprocessing

## Summary

A stepwise MALDI-TOF mass-spectrum preprocessing and inspection chain in R using MALDIquant: intensity (variance-stabilizing) transformation, smoothing, baseline removal, and peak detection, with each intermediate stage rendered as a panel for visual quality control, reproducing Figure 5 of Gatto et al. 2015.

## When to use

Use when a single MALDI-TOF (2D MS1-level) spectrum must be transformed, smoothed, baseline-corrected and peak-picked, and each preprocessing step needs to be visually inspected before downstream analysis.

## When NOT to use

- Input is already a peak list or feature table rather than a raw MALDI-TOF mass spectrum — the transformation/smoothing/baseline-correction chain would be meaningless
- Data comes from LC-MSMS peptide identification workflows (e.g. Pbase/mzID Proteins objects or mzID files) rather than 2D MS1 MALDI-TOF spectra
- You need to compare peptide detectability models across devices/datasets — the Pbase README warns 'you can't compare different devices/datasets (e.g. MALDI vs ESI)'

## Inputs

- Single MALDI-TOF mass spectrum (2D MS1-level), e.g. the demonstration MALDI-TOF spectrum shipped as example data with the MALDIquant R package

## Outputs

- Variance-stabilized (intensity-transformed) spectrum
- Smoothed, baseline-corrected spectrum flattened onto zero
- List of detected single peaks
- Multi-panel preprocessing figure (figure5_maldiquant_preprocessing_panels.png) showing raw, transformed, smoothed, baseline-corrected and peak-picked stages

## How to apply

Load a demonstration MALDI-TOF mass spectrum (e.g. the example data shipped with MALDIquant) into an R session as a MassSpectrum object. Apply the intensity transformation step first to stabilize variance across the mass range, then smooth the transformed spectrum and remove the chemical baseline so the signal is flattened onto zero, and finally run peak detection on the baseline-corrected spectrum to extract the list of single peaks. Render the successive stages — raw spectrum, transformed, smoothed, baseline-corrected, peak-picked — as a multi-panel figure using MALDIquant's traditional-graphics plotting functions, which are fast and flexible enough to investigate each single preprocessing step in detail (Fig. 5 of the article). Save the result as figure5_maldiquant_preprocessing_panels.png. The rationale is that MALDI-TOF raw spectra carry chemical noise and mass-range-dependent variance, so ordering variance stabilization before smoothing/baseline correction before peak detection yields interpretable single peaks rather than spurious calls.

## Related tools

- **MALDIquant** (R framework for processing and analysis of MALDI-TOF and other 2D MS1-level data; provides the intensity transformation, smoothing, baseline-correction, peak-detection and traditional-graphics plotting functions used in the Fig. 5 chain)
- **R** (Runtime environment in which the MALDIquant preprocessing and multi-panel plotting workflow is executed)
- **knitr** (Vignette engine used to render the R/Bioconductor visualization documentation that illustrates the workflow)
- **rmarkdown** (Vignette engine (knitr::rmarkdown) used alongside BiocStyle for the reproducible R document showing the plotting workflow)
- **BiocStyle** (Output styling (BiocStyle::html_document) for the reproducible vignette presenting the workflow)

## Examples

```
library(MALDIquant); s <- createMassSpectrum(mass, intensity); plot(transformIntensity(s, method="sqrt"))
```

## Evaluation signals

- The chemical baseline of the smoothed spectrum is flattened onto zero before peak detection is run
- All five successive stages (raw, transformed, smoothed, baseline-corrected, peak-picked) appear as separate panels in the saved PNG figure
- Peak detection is applied to the baseline-corrected spectrum only, not to the raw or transformed spectrum
- The rendered figure visually matches the structure of Figure 5 in Gatto et al. 2015 (each single preprocessing step inspectable in detail)
- The final file figure5_maldiquant_preprocessing_panels.png exists and was produced from the demonstration MALDI-TOF spectrum shipped with MALDIquant

## Limitations

- The workflow targets single-spectrum inspection; it is a step-by-step visual QC chain, not a high-throughput batch pipeline
- Peptide detectability models cannot be transferred across devices/datasets — per the Pbase README, MALDI vs ESI comparisons are not valid
- The article is a visualization overview; specific threshold values for smoothing or peak detection are not given in the provided text and must be chosen per dataset

## Evidence

- [full_text] MALDIquant plotting chain: [evidence span withheld — anchor status: not_found]
- [abstract] R plotting systems summary: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [readme] MALDI vs ESI comparability caveat: [evidence span withheld — anchor status: external_doc]
