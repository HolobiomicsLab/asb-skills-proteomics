---
name: proteins-object-construction-pbase
description: Use when you need to attach LC-MSMS peptide identifications (mzid) to
  protein sequences (fasta) as coordinate ranges to plot or explore protein-centric
  proteomics data in R.
when_to_use_negative:
- Input is raw MS spectra or a quantitative feature table rather than fasta + mzid
  identification files — Pbase's Proteins construction expects sequences plus mzIdentML
  identifications.
- Identifications are in a format unsupported by mzID/mzR rather than mzIdentML.
- The goal is quantitative MSnSet-level analysis; this skill only populates protein-centric
  sequence/range structures.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: Pbase
  role: Provides the central `Proteins` data object and methods to assemble sequences
    with peptide ranges and plot them
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: Biostrings
  role: Reads protein amino-acid sequences from the FASTA file feeding the Proteins
    object
- name: mzID
  role: Parses the mzIdentML file supplying experimentally observed peptides
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:b727e3b9449dd694f7015f1a9d2fafdf394bf0b07022b5399f91429d2ca7ef6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocStyle
  - Biostrings
  - Pbase
  - R
  - knitr
  - mzID
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Proteins object construction with Pbase

## Summary

Build Pbase `Proteins` instances by combining protein sequences read from a FASTA file (Biostrings) with peptide identifications parsed from an mzIdentML file (mzID), so peptides are mapped as coordinate ranges along protein sequences for downstream visualization.

## When to use

Use when you need to attach LC-MSMS peptide identifications (mzid) to protein sequences (fasta) as coordinate ranges to plot or explore protein-centric proteomics data in R.

## When NOT to use

- Input is raw MS spectra or a quantitative feature table rather than fasta + mzid identification files — Pbase's Proteins construction expects sequences plus mzIdentML identifications.
- Identifications are in a format unsupported by mzID/mzR rather than mzIdentML.
- The goal is quantitative MSnSet-level analysis; this skill only populates protein-centric sequence/range structures.

## Inputs

- Protein sequences in FASTA format (e.g. HUMAN_2015_02_selected.fasta, UniProt 2015_02 subset)
- Peptide identifications in mzIdentML (.mzid) format (e.g. Thermo_Hela_PRTC_selected.mzid from MSGF+)
- Protein identifiers used for subsetting (e.g. UniProt accessions A4UGR9, P00558)

## Outputs

- Pbase `Proteins` instances containing sequences plus peptide ranges-of-interest
- Per-protein plots of peptide ranges along sequences (e.g. Figure 6 panels for A4UGR9 and P00558)

## How to apply

Read protein sequences from the Pbase extdata FASTA (e.g. HUMAN_2015_02_selected.fasta, a restricted set of high scoring human proteins from UniProt release 2015_02) using Biostrings. Load peptide identifications from the mzIdentML file (e.g. Thermo_Hela_PRTC_selected.mzid, generated with MSGF+ on a Thermo Orbitrap Q Exactive dataset) with the mzID package. Assemble both into `Proteins` instances with Pbase, so each protein links to a set of ranges of interest (experimentally observed peptides) mapped as coordinates along the protein sequence. Once constructed, the object can be subset by protein identifier (e.g. A4UGR9, P00558) and rendered with Pbase's protein plotting methods, or overlaid with gene models (e.g. via ensembldb) for genomic coordinate views.

## Related tools

- **Pbase** (Provides the central `Proteins` data object and methods to assemble sequences with peptide ranges and plot them) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **Biostrings** (Reads protein amino-acid sequences from the FASTA file feeding the Proteins object)
- **mzID** (Parses the mzIdentML file supplying experimentally observed peptides)

## Examples

```
fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase"); idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package = "Pbase")
```

## Evaluation signals

- The Proteins object contains sequences matching the FASTA (e.g. the restricted UniProt 2015_02 human set) and non-empty peptide ranges for identified proteins.
- Peptide ranges fall within protein sequence coordinate bounds and align to the corresponding amino-acid subsequences.
- Rendering proteins A4UGR9 and P00558 reproduces Figure 6 panels: protein track plus peptide track, with correct accession labels.
- The P00558 coordinate view overlays correctly onto the ENSEMBL gene model without offset errors.
- All panels render and export successfully (e.g. as figure6_panels.png).

## Limitations

- Pbase was under heavy development and likely to considerably change, per the README; APIs may shift between versions.
- The demonstration data is a restricted set of high scoring human proteins from a single instrument (Thermo Orbitrap Q Exactive) and single search engine (MSGF+); generalization to other engines/datasets is not demonstrated here.
- Genomic coordinate mapping of proteins to genome sequence requires additional integration (ensembldb / mapping vignette) beyond basic Proteins construction.

## Evidence

- [intro] Proteins objects populated from fasta and mzid: "`Proteins` objects are populated by protein sequences stemming from a fasta file and the peptides typically originate from an LC-MSMS experiment."
- [intro] Peptides mapped as coordinates: "an optional set of peptides features that are mapped as coordinates along the proteins"
- [intro] Ranges of interest carry annotation: "Each protein links to a set of ranges of interest, such as protein domains of experimentally observed peptides"
- [intro] Restricted UniProt 2015_02 protein set: "A restricted set of high scoring human proteins from the UniProt release `2015_02` were searched using the `MSGF+` search engine."
- [readme] Interoperability with mzID and fasta-backed objects: [evidence span withheld — anchor status: external_doc]
