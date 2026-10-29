---
name: spectral-intensity-transformation
description: Use when you need to preprocess and inspect a single MALDI-TOF or other
  2D MS1-level mass spectrum in R, starting with intensity transformation to stabilize
  variance across the mass range before smoothing, baseline removal, and peak detection.
when_to_use_negative:
- Input is already a peak list or feature table — there is no raw intensity spectrum
  left to transform, smooth, or baseline-correct
- The data are LC-MS/MS peptide identifications handled via Pbase/mzID (Proteins objects,
  mzid files) rather than single 2D MS1-level spectra
- The goal is cross-device comparison of detection models — per the README, models
  can't be compared across devices/datasets (e.g. MALDI vs ESI)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3218
tools:
- name: MALDIquant
  role: Performs the intensity transformation, smoothing, baseline correction, peak
    detection, and traditional-graphics plotting of each preprocessing step on the
    MALDI-TOF spectrum
- name: R
  role: Runtime environment in which the MALDI-TOF demonstration spectrum is loaded
    and the preprocessing-to-visualization chain is executed
- name: knitr
  role: Reproducible-documentation engine used alongside rmarkdown and BiocStyle to
    render the demonstrated R workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:4665cbfa41a15bcfcbe11c805819bfabd117883405f90c3027fbfcd72c19b76e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MALDIquant
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spectral-intensity-transformation

## Summary

Variance-stabilizing intensity transformation of a single MALDI-TOF (2D MS1-level) mass spectrum in R with MALDIquant, performed as the first step of a per-step-inspectable preprocessing chain (transform → smooth → baseline-correct → peak-pick) rendered with MALDIquant's traditional-graphics plotting functions.

## When to use

Use when you need to preprocess and inspect a single MALDI-TOF or other 2D MS1-level mass spectrum in R, starting with intensity transformation to stabilize variance across the mass range before smoothing, baseline removal, and peak detection.

## When NOT to use

- Input is already a peak list or feature table — there is no raw intensity spectrum left to transform, smooth, or baseline-correct
- The data are LC-MS/MS peptide identifications handled via Pbase/mzID (Proteins objects, mzid files) rather than single 2D MS1-level spectra
- The goal is cross-device comparison of detection models — per the README, models can't be compared across devices/datasets (e.g. MALDI vs ESI)

## Inputs

- Raw MALDI-TOF mass spectrum (intensity vs. m/z), e.g. the demonstration single-spectrum example data shipped with MALDIquant
- R session with MALDIquant installed

## Outputs

- Variance-stabilized (transformed) spectrum
- Smoothed spectrum
- Baseline-corrected spectrum with signal flattened onto zero
- List of detected single peaks
- Multi-panel PNG figure (figure5_maldiquant_preprocessing_panels.png) showing every preprocessing stage

## How to apply

Load the demonstration MALDI-TOF mass spectrum shipped as example data with MALDIquant in an R session, and keep the raw spectrum as the reference panel. Apply MALDIquant's intensity transformation step to stabilize variance across the full mass range — the raw intensity noise level varies with m/z, so the transform must precede smoothing and peak detection. Then smooth the transformed spectrum, estimate and remove the chemical baseline so the signal is flattened onto zero, and run peak detection on the baseline-corrected spectrum to extract the list of single peaks. Render each successive preprocessing stage (raw, transformed, smoothed, baseline-corrected, peak-picked) as its own panel using MALDIquant's plotting functions, which are 'based on the traditional graphics infrastructure' and 'fast and flexible', allowing each single preprocessing step to be investigated in detail. Compose the panels into the multi-panel figure and save it as figure5_maldiquant_preprocessing_panels.png, reproducing the Fig. 5 chain.

## Related tools

- **MALDIquant** (Performs the intensity transformation, smoothing, baseline correction, peak detection, and traditional-graphics plotting of each preprocessing step on the MALDI-TOF spectrum)
- **R** (Runtime environment in which the MALDI-TOF demonstration spectrum is loaded and the preprocessing-to-visualization chain is executed)
- **knitr** (Reproducible-documentation engine used alongside rmarkdown and BiocStyle to render the demonstrated R workflow)

## Examples

```
Rscript figure5_maldiquant_preprocessing_panels.R  # loads MALDIquant's demo MALDI-TOF spectrum, transforms intensities to stabilize variance, smooths, baseline-corrects, peak-detects, and saves figure5_maldiquant_preprocessing_panels.png
```

## Evaluation signals

- The transformed spectrum shows stabilized variance across the entire mass range (noise level no longer scales with m/z as in the raw panel)
- After baseline correction, the chemical baseline is removed and the signal is flattened onto zero
- Peak detection on the baseline-corrected spectrum yields a concrete list of single peaks
- The saved PNG contains one panel per named stage — raw, transformed, smoothed, baseline-corrected, peak-picked — matching the Figure 5 chain
- Each panel is rendered with MALDIquant's own plotting functions, so every preprocessing step can be inspected individually

## Limitations

- The demonstration uses the single MALDI-TOF spectrum shipped with MALDIquant; the surrounding article is an overview of R plotting systems for MS-based proteomics, so this chain is a per-step inspection aid rather than a full pipeline for large batches
- Per the README's take-home-message on Mallick et al. 2007, results cannot be compared across different devices/datasets (e.g. MALDI vs ESI)

## Evidence

- [results] MALDIquant's plotting functions, based on the traditional graphics infrastructure, are fast and flexible and allow the user to investigate each single preprocessing step in detail: [evidence span withheld — anchor status: not_found]
- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [readme] take-home-message: The model of one species is comparable to another if the evolutionary distance is small (e.g. yeast and human) but you can't compare different devices/datasets (e.g. MALDI vs ESI): [evidence span withheld — anchor status: external_doc]
