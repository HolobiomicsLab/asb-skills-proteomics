---
name: precursor-charge-state-filtering
description: Use when one peptide ID appears on multiple MS2 spectra with different
  precursor charge states (e.g. z=2 vs z=3 in an iTRAQ 4-plex experiment) and the
  goal is a charge-resolved spectral overlay comparison.
when_to_use_negative:
- Only one spectrum exists for the peptide (no charge-state comparison is possible)
- Input is processed peptide/protein data (e.g. a Proteins object or MSnSet) rather
  than raw MS2 spectra in an MSnExp
- The goal is peptide-to-protein mapping rather than spectral fragment-ion comparison
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Pbase
  role: provides the bundled iTRAQ 4-plex MS2 example data and the Proteins/MSnExp
    integration used to locate and subset spectra by peptide ID
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: MSnbase
  role: MSnExp data abstraction that holds MS2 spectra with precursor charge metadata
    for subsetting and visualization
- name: protViz
  role: style of fragment-ion annotation (b- and y-type labels) applied to the overlaid
    spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:aae1a87f5a6715cfb20bdf676750a6585a705cbb386f4d440b278d2a53343d96
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - Pbase
  - R
  - protViz
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# precursor-charge-state-filtering

## Summary

Splits an MSnExp object of identified MS2 spectra by precursor charge state so that spectra of the same peptide (e.g. IMIDLDGTENK at z=2 and z=3 in an iTRAQ 4-plex run) can be overlaid and compared in a single annotated panel.

## When to use

Use when one peptide ID appears on multiple MS2 spectra with different precursor charge states (e.g. z=2 vs z=3 in an iTRAQ 4-plex experiment) and the goal is a charge-resolved spectral overlay comparison.

## When NOT to use

- Only one spectrum exists for the peptide (no charge-state comparison is possible)
- Input is processed peptide/protein data (e.g. a Proteins object or MSnSet) rather than raw MS2 spectra in an MSnExp
- The goal is peptide-to-protein mapping rather than spectral fragment-ion comparison

## Inputs

- iTRAQ 4-plex raw MS2 spectra bundled in the Pbase package extdata directory
- MSnExp instance (via Pbase/MSnbase readers)
- Peptide identification matching the target sequence IMIDLDGTENK
- Precursor charge-state metadata (z=2 and z=3 spectra)

## Outputs

- Charge-2 and charge-3 MS2 spectra of the same peptide, separated into groups
- Annotated overlay of both charge states with labelled b- and y-type fragment ions (protViz style)
- Saved figure panel (e.g. figure4_imidldgtenk_fragment_panel.png)

## How to apply

Load the Pbase/MSnbase stack in R and read the bundled iTRAQ 4-plex MS2 example file (package extdata) into an MSnExp instance. Subset the MSnExp to only the spectra whose peptide identification matches the target sequence (IMIDLDGTENK). Separate the retained spectra by their precursor charge state, keeping the charge-2 and charge-3 groups distinct. Overlay the two charge-state spectra and annotate b- and y-type fragment ions in the protViz style, reproducing the central panel of Figure 4. The rationale is that the MSnExp data abstraction preserves precursor charge metadata, enabling direct visual comparison of how charge state changes the fragment series of an identical peptide sequence.

## Related tools

- **Pbase** (provides the bundled iTRAQ 4-plex MS2 example data and the Proteins/MSnExp integration used to locate and subset spectra by peptide ID) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **MSnbase** (MSnExp data abstraction that holds MS2 spectra with precursor charge metadata for subsetting and visualization)
- **protViz** (style of fragment-ion annotation (b- and y-type labels) applied to the overlaid spectra)

## Evaluation signals

- The rendered panel visually matches the Figure 4 layout: an overlay of charge-2 and charge-3 spectra of IMIDLDGTENK with labelled fragments
- Exactly two spectra are retained after subsetting, one per precursor charge state (z=2 and z=3)
- All retained spectra share the same peptide identification (IMIDLDGTENK)
- b- and y-type fragment ions are annotated in the protViz style on the overlay
- Output file is produced as figure4_imidldgtenk_fragment_panel.png and confirmed by visual inspection against the published figure

## Limitations

- Pbase is under heavy development and is likely to considerably change, so API details may shift between versions
- Demonstrated on a single bundled iTRAQ 4-plex example; adaptation to other instruments (e.g. Thermo Orbitrap Q Exactive) or acquisition types is not covered by this figure workflow
- Validation is by visual inspection against the published figure, not an automated metric

## Evidence

- [abstract] abstract overview of R plotting for raw MS data: "We present a summary of R's plotting systems and how they are used to visualize and understand raw and processed MS-based proteomics data."
- [readme] MSnExp interoperability: "protein identification results (`mzID` or (devel) `mzR` packages) and possibly
also `MSnExp` and `MSnSet` instances."
- [intro] peptides originate from LC-MSMS experiments: "`Proteins` objects are populated by protein sequences stemming from a
fasta file and the peptides typically originate from an LC-MSMS
experiment."
- [intro] extdata pattern for bundled Pbase data: "fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta",
                      package = "Pbase")"
