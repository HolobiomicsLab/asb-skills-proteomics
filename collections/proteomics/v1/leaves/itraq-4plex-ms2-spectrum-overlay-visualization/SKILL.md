---
name: itraq-4plex-ms2-spectrum-overlay-visualization
description: Use when raw MS2 spectra of one identified peptide exist at multiple
  precursor charge states (e.g. iTRAQ 4-plex) and the goal is an overlaid, fragment-annotated
  comparison plot in R.
when_to_use_negative:
- Input is already a processed/quantitative matrix (e.g. an MSnSet feature table)
  rather than raw MS2 spectra — the skill operates on the MSnExp raw-data abstraction
- Only one charge state of the peptide is available, so the charge-2 vs charge-3 overlay
  comparison is impossible
- The goal is protein/peptide mapping against FASTA sequences (the Pbase Proteins
  object workflow with HUMAN_2015_02_selected.fasta and Thermo_Hela_PRTC_selected.mzid)
  rather than raw spectrum visualization
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Pbase
  role: Host package whose extdata provides the iTRAQ 4-plex MS2 example file and
    whose readers populate the MSnExp instance for visualization
  repo: https://github.com/lgatto/Pbase
- name: MSnbase
  role: Provides the MSnExp data abstraction that stores and enables direct comparison
    of the two MS2 spectra at charge states 2 and 3
- name: protViz
  role: Supplies the annotation style used to label b- and y-type fragment ions on
    the overlaid spectra
- name: R
  role: Runtime environment in which the plotting and overlay workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:0480128f5d9b2ad29fd7c8e55f16a23d925c00be1365749e31dba6c8808acd73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lgatto/Pbase
  tools:
  - MSnbase
  - Pbase
  - R
  - protViz
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# itraq-4plex-ms2-spectrum-overlay-visualization

## Summary

Uses R/Bioconductor (Pbase/MSnbase data abstraction plus protViz-style annotation) to overlay two MS2 spectra of the same peptide — IMIDLDGTENK from an iTRAQ 4-plex experiment — at precursor charge states 2 and 3, reproducing the central panel of Figure 4 of Gatto et al. 2015.

## When to use

Use when raw MS2 spectra of one identified peptide exist at multiple precursor charge states (e.g. iTRAQ 4-plex) and the goal is an overlaid, fragment-annotated comparison plot in R.

## When NOT to use

- Input is already a processed/quantitative matrix (e.g. an MSnSet feature table) rather than raw MS2 spectra — the skill operates on the MSnExp raw-data abstraction
- Only one charge state of the peptide is available, so the charge-2 vs charge-3 overlay comparison is impossible
- The goal is protein/peptide mapping against FASTA sequences (the Pbase Proteins object workflow with HUMAN_2015_02_selected.fasta and Thermo_Hela_PRTC_selected.mzid) rather than raw spectrum visualization

## Inputs

- iTRAQ 4-plex raw MS2 spectra file bundled in Pbase extdata (raw MS-level data from a Thermo Orbitrap Q Exactive acquisition per the article's example data)
- Peptide identification matching IMIDLDGTENK within the MSnExp instance
- Precursor charge-state metadata (charge 2 and charge 3 spectra)

## Outputs

- Overlaid plot of the charge-2 and charge-3 MS2 spectra of IMIDLDGTENK with b- and y-type fragment ion annotations in protViz style
- figure4_imidldgtenk_fragment_panel.png (saved figure panel)

## How to apply

Load the Pbase package in R and locate the iTRAQ 4-plex MS2 example file bundled in the package extdata directory. Read the raw MS2 spectra into an MSnExp instance using the Pbase/MSnbase readers, then subset the MSnExp to the spectra whose peptide identification matches IMIDLDGTENK. Separate the retained spectra by precursor charge state 2 and charge state 3, overlay the two spectra, and annotate the b- and y-type fragment ions in the protViz style to rebuild the Figure 4 central panel. Save the finished panel as figure4_imidldgtenk_fragment_panel.png; correctness is judged by visual comparison of the rendered overlay and fragment labels against the published Figure 4 layout.

## Related tools

- **Pbase** (Host package whose extdata provides the iTRAQ 4-plex MS2 example file and whose readers populate the MSnExp instance for visualization) — https://github.com/lgatto/Pbase
- **MSnbase** (Provides the MSnExp data abstraction that stores and enables direct comparison of the two MS2 spectra at charge states 2 and 3)
- **protViz** (Supplies the annotation style used to label b- and y-type fragment ions on the overlaid spectra)
- **R** (Runtime environment in which the plotting and overlay workflow is executed)

## Examples

```
library("Pbase"); iTRAQ4 <- readMSnExp(system.file("extdata", package = "Pbase"))
```

## Evaluation signals

- The rendered panel contains an overlay of both the charge-2 and charge-3 MS2 spectra of IMIDLDGTENK, matching the central panel layout of published Figure 4
- b- and y-type fragment ions are labelled on the overlay in the protViz annotation style
- Spectra were subset from an MSnExp instance by peptide identification (IMIDLDGTENK) before plotting, not manually extracted
- Output file figure4_imidldgtenk_fragment_panel.png exists and passes visual inspection against the published figure

## Limitations

- Pbase is 'under heavy development and is likely to considerably change in the near future', so the example extdata file location and reader API may drift between versions
- The workflow depends on the specific iTRAQ 4-plex example dataset bundled in Pbase extdata; other raw files require an identified multi-charge peptide to be present
- Validation is by visual inspection against the published figure rather than a programmatic assertion

## Evidence

- [abstract] abstract: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [intro] intro: "This vignette briefly introduces the central data object of the `Pbase` package, namely `Proteins` instances"
- [readme] readme: "Manipulating and exploring protein and proteomics data."
- [readme] readme-interoperability: "The package allows to easily interact with `AAString` and
`AAStringSet` instances, protein databases such as UniProt (and
possibly biomaRt in the future) using protein identifiers,"
