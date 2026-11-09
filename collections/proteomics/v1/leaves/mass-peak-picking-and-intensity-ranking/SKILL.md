---
name: mass-peak-picking-and-intensity-ranking
description: Use when a mass spectrometry imaging dataset contains one MassSpectrum
  per pixel and you need a summarized mean spectrum plus spatial ion-image slices
  centered on the most intense peaks.
when_to_use_negative:
- Input is already a processed feature table or peak matrix rather than per-pixel
  raw spectra — no averaging or peak picking is needed
- The data are LC-MS/MS peptide identifications (e.g. mzID results mapped to Proteins
  objects in Pbase) rather than MALDI imaging spectra
- You need peak windows narrower or wider than exactly 2 Da; this skill's validation
  criterion requires each slice window to span exactly 2 Da
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MALDIquant
  role: Imports per-pixel MassSpectrum objects, averages spectra into a mean spectrum,
    performs peak picking, and supplies m/z-window extraction for the 2 Da ion-image
    slices
- name: Cardinal
  role: Optional cross-check of the ion-image rendering of the 2 Da slices
- name: R
  role: Environment in which the MALDIquant imaging summarization and plotting workflow
    is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:a21838dc412bfa14e1b8b50d2e4df0b0fe26c94c444a0f1e479d5bdd27a09742
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MALDIquant
  - R
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mass-peak-picking-and-intensity-ranking

## Summary

A MALDIquant-based workflow that summarizes a per-pixel MALDI mass spectrometry imaging dataset (e.g. the mouse kidney demonstration dataset) into a mean spectrum, picks peaks from it, and ranks peaks by intensity to select the three highest peaks for 2 Da ion-image slices.

## When to use

Use when a mass spectrometry imaging dataset contains one MassSpectrum per pixel and you need a summarized mean spectrum plus spatial ion-image slices centered on the most intense peaks.

## When NOT to use

- Input is already a processed feature table or peak matrix rather than per-pixel raw spectra — no averaging or peak picking is needed
- The data are LC-MS/MS peptide identifications (e.g. mzID results mapped to Proteins objects in Pbase) rather than MALDI imaging spectra
- You need peak windows narrower or wider than exactly 2 Da; this skill's validation criterion requires each slice window to span exactly 2 Da

## Inputs

- MALDI imaging dataset with one MassSpectrum object per pixel (MALDIquant MassSpectrum objects)
- Mouse kidney MALDI imaging demonstration dataset (imported via MALDIquant import functions)

## Outputs

- Single mean MassSpectrum collapsed across all imaging pixels
- Ranked peak list from the mean spectrum (top three peaks by intensity)
- Three 2 Da ion-image slices, each spanning ±1 Da around one of the three most intense peaks, rendered over the tissue section

## How to apply

Load the mouse kidney MALDI imaging dataset into R with MALDIquant import functions so each imaging pixel becomes a MassSpectrum object. Collapse all per-pixel spectra into a single mean spectrum using MALDIquant's spectrum-averaging routine (averageMassSpectra) across every pixel of the tissue section. Run peak picking (detectPeaks) on the mean spectrum and rank the detected peaks by intensity to identify the three highest peaks. For each of the three top peaks, extract a 2 Da m/z window (±1 Da around the peak centroid) and plot the corresponding spatial ion-image slices over the kidney tissue section; optionally cross-check the rendering with the Cardinal imaging package.

## Related tools

- **MALDIquant** (Imports per-pixel MassSpectrum objects, averages spectra into a mean spectrum, performs peak picking, and supplies m/z-window extraction for the 2 Da ion-image slices)
- **Cardinal** (Optional cross-check of the ion-image rendering of the 2 Da slices)
- **R** (Environment in which the MALDIquant imaging summarization and plotting workflow is executed)

## Examples

```
avg <- averageMassSpectra(msiPixels); peaks <- detectPeaks(avg); top3 <- peaks[order(unlist(lapply(intensity(peaks), max)), decreasing=TRUE)][1:3]
```

## Evaluation signals

- The mean spectrum renders as a single spectrum collapsed from all imaging pixels, not one spectrum per pixel
- Each ion-image slice window spans exactly 2 Da, centered (±1 Da) on one peak centroid
- Exactly three slices are produced, corresponding to the three highest-intensity peaks of the mean spectrum
- The combined layout (mean spectrum plus three 2 Da slices) reproduces the layout and content of Figure 7 of the mouse kidney demonstration
- Peak ranking is by intensity on the mean spectrum's picked peaks, not on individual pixel spectra

## Limitations

- Cross-checking with the Cardinal imaging package is optional, so renderings produced only by MALDIquant plotting are not independently validated
- Selecting only the three highest peaks is a demonstration-level choice; lower-abundance analytes of biological interest may be missed
- MALDI vs ESI dataset characteristics differ, so detectability models/rankings derived from one dataset or device type should not be transferred to another (per the referenced Mallick et al. 2007 take-home-message in the README)

## Evidence

- [full_text] mean spectrum plus three slices: [evidence span withheld — anchor status: not_found]
- [full_text] 2 Da window extraction: [evidence span withheld — anchor status: not_found]
- [full_text] spectrum averaging: [evidence span withheld — anchor status: not_found]
- [abstract] role of visualization: "Data visualization plays a key role in high-throughput biology. It is an essential tool for data exploration allowing to shed light on data structure and patterns of interest."
- [readme] MALDI vs ESI transfer caveat: [evidence span withheld — anchor status: external_doc]
