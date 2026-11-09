---
name: single-peak-detection-from-mass-spectra
description: Use when working with raw MALDI-TOF or other 2D MS1-level spectra in
  R and you need to inspect every preprocessing step (transformation, smoothing, baseline
  correction, peak picking) in detail.
when_to_use_negative:
- Input is peptide identification results or protein sequences to explore at the protein
  level — use Pbase's Proteins objects instead
- Data are LC-MSMS identification files (mzID) or protein FASTA files rather than
  raw MS1 spectra
- You need a static figure only, with no inspection of intermediate preprocessing
  stages
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MALDIquant
  role: Provides the plotting and preprocessing functions (intensity transformation,
    smoothing, baseline removal, peak detection) applied step-by-step to the single
    spectrum
- name: R
  role: Execution environment for the MALDIquant preprocessing and multi-panel figure
    rendering
- name: knitr
  role: Vignette engine used to render the documented R/Bioconductor workflow
- name: rmarkdown
  role: Vignette rendering backend alongside knitr for the workflow documentation
- name: BiocStyle
  role: Document styling used for the published R/Bioconductor vignette output
- name: Pbase
  role: Related Bioconductor package for downstream protein/peptide-level exploration,
    interoperating with MSnExp/MSnSet instances
  repo: github.com/lgatto/Pbase
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:c7e16c7141c5c6dfbba04201d93bad4561c62acbdaaee5bf0b1e3ed838f124a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - MALDIquant
  - Pbase
  - R
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Single peak detection from mass spectra

## Summary

A MALDIquant-based R workflow for preprocessing a single MALDI-TOF mass spectrum — intensity transformation, smoothing, baseline removal, peak detection — with per-step visual inspection, as illustrated by Fig. 5 of the R/Bioconductor proteomics visualization overview.

## When to use

Use when working with raw MALDI-TOF or other 2D MS1-level spectra in R and you need to inspect every preprocessing step (transformation, smoothing, baseline correction, peak picking) in detail.

## When NOT to use

- Input is peptide identification results or protein sequences to explore at the protein level — use Pbase's Proteins objects instead
- Data are LC-MSMS identification files (mzID) or protein FASTA files rather than raw MS1 spectra
- You need a static figure only, with no inspection of intermediate preprocessing stages

## Inputs

- Single MALDI-TOF mass spectrum (mass/intensity pairs), e.g. MALDIquant example demo spectrum
- 2D MS1-level mass spectral data in R

## Outputs

- Variance-stabilized (intensity-transformed) spectrum
- Smoothed spectrum
- Baseline-corrected spectrum (signal flattened onto zero)
- List of detected single peaks
- Multi-panel preprocessing figure saved as figure5_maldiquant_preprocessing_panels.png

## How to apply

In an R session, load the demonstration MALDI-TOF mass spectrum shipped as example data with MALDIquant, then apply the intensity transformation step to stabilize variance across the mass range. Smooth the transformed spectrum and remove the chemical baseline so the signal is flattened onto zero, then run peak detection on the baseline-corrected spectrum to extract the list of single peaks. Render each stage (raw spectrum, transformed, smoothed, baseline-corrected, peak-picked) as a multi-panel figure, because MALDIquant's traditional-graphics plotting functions are deliberately fast and flexible so each single preprocessing step can be investigated in detail. Judge success by the flattened baseline and by a plotted peak list that aligns with visible signal after correction.

## Related tools

- **MALDIquant** (Provides the plotting and preprocessing functions (intensity transformation, smoothing, baseline removal, peak detection) applied step-by-step to the single spectrum)
- **R** (Execution environment for the MALDIquant preprocessing and multi-panel figure rendering)
- **knitr** (Vignette engine used to render the documented R/Bioconductor workflow)
- **rmarkdown** (Vignette rendering backend alongside knitr for the workflow documentation)
- **BiocStyle** (Document styling used for the published R/Bioconductor vignette output)
- **Pbase** (Related Bioconductor package for downstream protein/peptide-level exploration, interoperating with MSnExp/MSnSet instances) — github.com/lgatto/Pbase

## Examples

```
Rscript figure5_maldiquant_preprocessing_panels.R  # loads MALDIquant demo spectrum, transforms, smooths, removes baseline, detects peaks, renders 5 panels
```

## Evaluation signals

- Each preprocessing stage (raw, transformed, smoothed, baseline-corrected, peak-picked) renders successfully as its own panel
- After baseline removal, the chemical baseline is flattened so the signal sits on zero
- The intensity transformation stabilizes variance across the full mass range (no heteroscedastic spread of noise)
- Peak detection returns a non-empty list of single peaks from the baseline-corrected spectrum
- Final multi-panel figure is written to figure5_maldiquant_preprocessing_panels.png and reproduces the Fig. 5 layout

## Limitations

- MALDIquant's plotting is based on the traditional graphics infrastructure — fast and flexible, but not interactive per-step tuning
- The workflow is demonstrated on a single demonstration MALDI-TOF spectrum, not batch/high-throughput spectra processing
- Per Mallick et al. 2007 in the Pbase README, models/datasets should not be compared across devices (e.g. MALDI vs ESI) without care
- The enriched article index contains no discussion-section text, so caveats are drawn mainly from the workflow description and README

## Evidence

- [full_text] MALDIquant plotting functions allow per-step inspection: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: R plotting systems for raw and processed MS data: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [readme] README: cannot compare across devices (MALDI vs ESI): [evidence span withheld — anchor status: external_doc]
- [readme] README: Pbase interoperability context for related R MS tooling: [evidence span withheld — anchor status: external_doc]
