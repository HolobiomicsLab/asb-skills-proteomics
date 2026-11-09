---
name: peptide-to-protein-coordinate-mapping
description: Use when you have protein sequences from a fasta file and peptide identifications
  (e.g. mzIdentML from an LC-MSMS search) and need to place peptides as coordinate
  ranges on proteins for plotting or annotation.
when_to_use_negative:
- Input is already a feature/quantification table without sequence-level fasta or
  mzid identification data — coordinate mapping requires fasta sequences plus mzIdentML
  identifications.
- The goal is raw MS spectra visualization rather than protein-centric peptide placement
  — Pbase Proteins are populated from fasta and LC-MSMS identifications, not raw instrument
  files.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Pbase
  role: Provides the central Proteins data object that links protein sequences to
    ranges of interest (experimentally observed peptides) mapped as coordinates, and
    the protein plotting method used to render them
  repo: https://github.com/lgatto/Pbase
- name: Biostrings
  role: Reads the protein sequences from the fasta file before assembly into Proteins
    instances
- name: mzID
  role: Parses the mzIdentML peptide identification file used to populate peptide
    ranges
- name: knitr/rmarkdown
  role: Vignette engine used to render the documented workflow reproducibly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:465fd08a16b7c6906203298161a0c82d5bc9edde0bc1b9a1c58cc3ed643d64d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lgatto/Pbase
  tools:
  - BiocStyle
  - Biostrings
  - Pbase
  - R
  - knitr
  - knitr/rmarkdown
  - mzID
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# peptide-to-protein-coordinate-mapping

## Summary

Maps experimentally observed peptides as genomic/range coordinates along protein sequences using Pbase's Proteins data object, so that peptide identifications from an mzIdentML file can be visualized aligned to their parent protein sequences (e.g. the Figure 6 panels for A4UGR9 and P00558).

## When to use

Use when you have protein sequences from a fasta file and peptide identifications (e.g. mzIdentML from an LC-MSMS search) and need to place peptides as coordinate ranges on proteins for plotting or annotation.

## When NOT to use

- Input is already a feature/quantification table without sequence-level fasta or mzid identification data — coordinate mapping requires fasta sequences plus mzIdentML identifications.
- The goal is raw MS spectra visualization rather than protein-centric peptide placement — Pbase Proteins are populated from fasta and LC-MSMS identifications, not raw instrument files.

## Inputs

- Protein sequences in fasta format (e.g. HUMAN_2015_02_selected.fasta from Pbase extdata)
- Peptide identifications in mzIdentML format (e.g. Thermo_Hela_PRTC_selected.mzid from Pbase extdata)
- Optional gene model from an EnsemblDb resource for genome-coordinate overlay

## Outputs

- Pbase Proteins instances with peptides mapped as coordinate ranges along each protein
- Plotted protein/peptide track panels (e.g. saved as figure6_panels.png) showing peptide ranges aligned to protein sequences

## How to apply

Read protein sequences from a fasta file (e.g. Pbase extdata HUMAN_2015_02_selected.fasta, a restricted set of high-scoring human proteins from UniProt release 2015_02) using Biostrings. Load peptide identifications from the mzIdentML file (e.g. Thermo_Hela_PRTC_selected.mzid, produced by searching Thermo Orbitrap Q Exactive data with MSGF+) with the mzID package. Assemble fasta sequences and peptides into Pbase Proteins instances, where each protein links to a set of ranges of interest (experimentally observed peptides) that are mapped as coordinates along the proteins. Then plot a chosen protein (e.g. A4UGR9 or P00558) with its peptide ranges using Pbase's protein plotting method, optionally overlaying an ENSEMBL gene model for the coordinate view. Validate that rendered panels show the correct proteins and that peptide ranges align to the protein sequences.

## Related tools

- **Pbase** (Provides the central Proteins data object that links protein sequences to ranges of interest (experimentally observed peptides) mapped as coordinates, and the protein plotting method used to render them) — https://github.com/lgatto/Pbase
- **Biostrings** (Reads the protein sequences from the fasta file before assembly into Proteins instances)
- **mzID** (Parses the mzIdentML peptide identification file used to populate peptide ranges)
- **knitr/rmarkdown** (Vignette engine used to render the documented workflow reproducibly)

## Examples

```
library("Biostrings"); library("mzID"); library("Pbase"); fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase"); idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package = "Pbase")
```

## Evaluation signals

- All rendered panels display the correct proteins (A4UGR9 and P00558) as top tracks with identified peptides as second tracks
- Peptide ranges align correctly to the protein sequences read from HUMAN_2015_02_selected.fasta
- Proteins instances contain one set of ranges of interest per protein, with peptides originating from the mzID-parsed Thermo_Hela_PRTC_selected.mzid file
- Panels are successfully saved (e.g. figure6_panels.png) without mapping errors

## Limitations

- Pbase is under heavy development and likely to change considerably, so API details may shift between versions
- The demonstrated dataset is a restricted set of high-scoring human proteins from UniProt 2015_02 searched with MSGF+; other fasta/mzid sources may require adaptation
- The README notes take-home-message that detectability models do not transfer between species/datasets — but more directly for this skill, some interoperability (e.g. mzR, biomaRt) is marked as 'possibly'/'future'

## Evidence

- [intro] Proteins instances contain protein sequences with annotation data, each linking to ranges of interest (e.g. protein domains or experimentally observed peptides) that carry their own annotation: "Each protein links to a set of ranges of interest, such as protein domains of experimentally observed peptides"
- [intro] Proteins objects populated from fasta and LC-MSMS peptides: "`Proteins` objects are populated by protein sequences stemming from a fasta file and the peptides typically originate from an LC-MSMS experiment."
- [intro] Peptides mapped as coordinates along proteins: "an optional set of peptides features that are mapped as coordinates along the proteins"
- [intro] Extdata file locations for fasta and mzid: "fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase")"
- [readme] Interoperability with AAString and mzID identification results: [evidence span withheld — anchor status: external_doc]
