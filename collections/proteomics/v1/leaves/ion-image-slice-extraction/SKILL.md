---
name: ion-image-slice-extraction
description: Use when an MS imaging dataset (one MassSpectrum object per tissue pixel)
  must be summarized into a mean spectrum and rendered as spatial ion-image slices
  at selected m/z values, e.g. reproducing the mouse kidney MALDI imaging Figure 7
  layout.
when_to_use_negative:
- Input is LC-MS/MS peptide/protein identification data (e.g. fasta plus mzID results
  assembled into Proteins objects) rather than per-pixel imaging spectra — use the
  Pbase Proteins workflow instead
- The dataset is a single bulk spectrum with no spatial pixel coordinates, so ion-images
  cannot be formed
- Input is already an aggregated feature/intensity table rather than raw per-pixel
  MassSpectrum objects
edam_operation: http://edamontology.org/operation_0337
tools:
- name: MALDIquant
  role: Imports per-pixel MassSpectrum objects, collapses them into a mean spectrum
    via its spectrum-averaging routine, and picks/intensity-ranks peaks to define
    the 2 Da slice windows
- name: Cardinal
  role: Optional cross-check of the rendered ion-image slices and figure layout
- name: R
  role: Environment in which the MALDIquant imaging workflow (import, averaging, peak
    picking, slice plotting) is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:7e59950eac0161352eb12d15eb14914911de68df8d54ab65ce90c01c7e95c25a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MALDIquant
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# ion-image-slice-extraction

## Summary

Summarize a MALDI mass spectrometry imaging dataset — one spectrum per pixel across a tissue section — as a single mean spectrum, then visualize spatial ion-image slices of 2 Da around the three highest-intensity peaks, reproducing the mouse kidney Figure 7 demonstration using MALDIquant in R.

## When to use

Use when an MS imaging dataset (one MassSpectrum object per tissue pixel) must be summarized into a mean spectrum and rendered as spatial ion-image slices at selected m/z values, e.g. reproducing the mouse kidney MALDI imaging Figure 7 layout.

## When NOT to use

- Input is LC-MS/MS peptide/protein identification data (e.g. fasta plus mzID results assembled into Proteins objects) rather than per-pixel imaging spectra — use the Pbase Proteins workflow instead
- The dataset is a single bulk spectrum with no spatial pixel coordinates, so ion-images cannot be formed
- Input is already an aggregated feature/intensity table rather than raw per-pixel MassSpectrum objects

## Inputs

- MALDI MS imaging dataset containing one mass spectrum per pixel across a tissue section
- Per-pixel MassSpectrum objects as handled by MALDIquant's import functions in R

## Outputs

- Single mean spectrum averaged across all imaging pixels
- Peak list from the mean spectrum ranked by intensity (top three peaks identified)
- Three spatial ion-image slices, each spanning exactly 2 Da (±1 Da) centered on one of the three most intense peaks, plotted over the kidney tissue section

## How to apply

Load the mouse kidney MALDI imaging dataset into R with MALDIquant's import functions so that each imaging pixel yields a MassSpectrum object. Collapse the per-pixel spectra into a single mean spectrum using MALDIquant's spectrum-averaging routine across all imaging pixels, which summarizes the whole tissue section in one trace. Pick peaks from the mean spectrum and rank them by intensity, retaining the three highest peaks. For each of those three peaks, extract a 2 Da m/z window (±1 Da around the peak) and plot the corresponding spatial ion-image slices over the kidney tissue section, showing where each ion localizes. Optionally cross-check the ion-image rendering with the Cardinal imaging package to confirm the visual layout.

## Related tools

- **MALDIquant** (Imports per-pixel MassSpectrum objects, collapses them into a mean spectrum via its spectrum-averaging routine, and picks/intensity-ranks peaks to define the 2 Da slice windows)
- **Cardinal** (Optional cross-check of the rendered ion-image slices and figure layout)
- **R** (Environment in which the MALDIquant imaging workflow (import, averaging, peak picking, slice plotting) is executed)

## Evaluation signals

- The mean spectrum plus three 2 Da ion-image slices render and together reproduce the layout and content of Figure 7
- Each slice window spans exactly 2 Da and is centered on one of the three most intense peaks of the mean spectrum
- Peak ranking by intensity identifies the same three top peaks on the averaged spectrum
- Averaging the per-pixel MassSpectrum objects yields one representative trace rather than pixel-wise traces
- Optional Cardinal rendering of the same slices agrees with the MALDIquant-based plots

## Limitations

- The demonstrated selection rule is limited to the three highest peaks with a fixed ±1 Da (2 Da total) window; other numbers of peaks or window widths are not covered by the validated workflow
- Ion-image rendering itself is best cross-checked with the Cardinal imaging package, indicating MALDIquant alone may not fully cover the spatial visualization step
- The workflow is demonstrated on the mouse kidney MALDI imaging demo dataset; transfer to other tissues or instruments is not validated in the source card

## Evidence

- [other] The mouse kidney MALDI imaging demonstration dataset is summarized by a mean spectrum and three slices covering a range of 2 Da around the three highest peaks.: [evidence span withheld — anchor status: not_found]
- [other] For each of the three highest peaks, extract a 2 Da m/z window (±1 Da around the peak) and plot the corresponding spatial ion-image slices over the kidney tissue section.: [evidence span withheld — anchor status: not_found]
- [abstract] We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data.: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
