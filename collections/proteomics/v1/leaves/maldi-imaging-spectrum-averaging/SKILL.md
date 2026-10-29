---
name: maldi-imaging-spectrum-averaging
description: Use when the input is an MS imaging acquisition (a mass spectrum per
  pixel across a tissue section, e.g. the mouse kidney MALDI imaging demonstration
  dataset) and the goal is to summarize it as a mean spectrum plus spatial ion-image
  slices around its highest peaks in R.
when_to_use_negative:
- The input is peptide/protein identification or LC-MS/MS sequence data (e.g. FASTA
  plus mzID PSM results) — that is the scope of Pbase's Proteins objects, not MS imaging
  summarization
- The data is already a processed feature/peak table with no per-pixel spectra, so
  averaging raw pixels is impossible
- The goal is single-spectrum visualization (no spatial pixel dimension), where peak
  picking on one MassSpectrum suffices and ion images are undefined
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Analysis environment in which the imaging spectra are loaded, averaged, and
    plotted
- name: MALDIquant
  role: Imports per-pixel MassSpectrum objects, averages spectra into a mean spectrum,
    and picks/ranks peaks
- name: Cardinal
  role: Optional cross-check of the ion-image slice rendering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:d5d8bec3844c5050a085d6e3361e58d3598fbc4839cd0bce89ef7c368c9eee33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MALDIquant
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# maldi-imaging-spectrum-averaging

## Summary

Summarizes an MS imaging dataset (one spectrum per tissue-section pixel) in R by collapsing all per-pixel MassSpectrum objects into a single mean spectrum with MALDIquant, then rendering 2-Da m/z ion-image slices around the three most intense peaks, as demonstrated for the mouse kidney MALDI imaging data (Fig 7 layout).

## When to use

Use when the input is an MS imaging acquisition (a mass spectrum per pixel across a tissue section, e.g. the mouse kidney MALDI imaging demonstration dataset) and the goal is to summarize it as a mean spectrum plus spatial ion-image slices around its highest peaks in R.

## When NOT to use

- The input is peptide/protein identification or LC-MS/MS sequence data (e.g. FASTA plus mzID PSM results) — that is the scope of Pbase's Proteins objects, not MS imaging summarization
- The data is already a processed feature/peak table with no per-pixel spectra, so averaging raw pixels is impossible
- The goal is single-spectrum visualization (no spatial pixel dimension), where peak picking on one MassSpectrum suffices and ion images are undefined

## Inputs

- MS imaging dataset: one mass spectrum per pixel across a tissue section (e.g. mouse kidney MALDI imaging demonstration data)
- Per-pixel MassSpectrum objects loaded via MALDIquant import functions

## Outputs

- A single mean spectrum averaged across all imaging pixels
- Ranked peak list from the mean spectrum (top three peaks by intensity)
- Three 2-Da ion-image slices (±1 Da around each top peak) plotted over the tissue section

## How to apply

Import the imaging run with MALDIquant's import functions so each pixel becomes a MassSpectrum object. Collapse all per-pixel spectra into one representative trace using MALDIquant's spectrum-averaging routine (mean method) across all imaging pixels. Pick peaks on the mean spectrum and rank them by intensity to identify the three highest peaks. For each of the three highest peaks, extract a 2-Da m/z window (±1 Da around the peak) and plot the corresponding spatial ion-image slices over the tissue section. Optionally cross-check the rendering with the Cardinal imaging package.

## Related tools

- **R** (Analysis environment in which the imaging spectra are loaded, averaged, and plotted)
- **MALDIquant** (Imports per-pixel MassSpectrum objects, averages spectra into a mean spectrum, and picks/ranks peaks)
- **Cardinal** (Optional cross-check of the ion-image slice rendering)

## Examples

```
meanSpec <- averageMassSpectra(pixelSpectra, method = "mean")
```

## Evaluation signals

- The mean spectrum is a single trace computed across all imaging pixels, not a subset or per-region average
- Exactly three ion-image slices are produced, each corresponding to one of the three most intense peaks of the mean spectrum
- Each slice window spans exactly 2 Da, centered on its peak (±1 Da)
- The mean spectrum plus three slices reproduce the layout and content of the mouse kidney Fig 7 demonstration
- A Cardinal re-rendering of the same slices matches the MALDIquant-based output

## Limitations

- Validation is visual: correctness is judged by reproducing the Fig 7 layout and content of the demonstration dataset, not by a numeric metric
- Slice width is fixed at 2 Da around the top-intensity peaks; peaks ranked by intensity may not be the biologically most relevant m/z for a given tissue

## Evidence

- [other] The mouse kidney MALDI imaging demonstration dataset is summarized by a mean spectrum and three slices covering a range of 2 Da around the three highest peaks.: [evidence span withheld — anchor status: not_found]
- [other] Collapse the per-pixel spectra into a single mean spectrum using MALDIquant's spectrum-averaging routine across all imaging pixels.: [evidence span withheld — anchor status: not_found]
- [other] For each of the three highest peaks, extract a 2 Da m/z window (±1 Da around the peak) and plot the corresponding spatial ion-image slices over the kidney tissue section.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we provided a short overview of the application of the R software to the visualization of proteomics data.: "Here, we provided a short overview of the application of the R software to the visualization of proteomics data."
