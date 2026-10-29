---
name: protein-sequence-loading-from-fasta
description: Use when protein sequences live in a fasta file (e.g. HUMAN_2015_02_selected.fasta
  from UniProt 2015_02) and peptide identifications from an mzIdentML file (e.g. Thermo_Hela_PRTC_selected.mzid)
  must be combined into Proteins objects for plotting.
when_to_use_negative:
- Peptide identifications are in a raw search-engine output format rather than mzIdentML
  — Pbase interoperates with identification results via the mzID or (devel) mzR packages,
  but the documented workflow here reads mzid files
- The goal is protein domain annotation rather than observed peptides — protein domains
  come from the EnsemblDb integration (Pbase-with-ensembldb vignette), not fasta+mzid
  loading
- The data is already a quantitative matrix (e.g. MSnSet) rather than sequence-level
  fasta/mzid inputs
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0077
tools:
- name: Pbase
  role: Provides the `Proteins` central data object; sequences from a fasta file and
    peptides from an LC-MSMS experiment (mzid) are assembled into Proteins instances
    with peptides mapped as coordinates along proteins
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: Biostrings
  role: Reads the protein sequences from the fasta file (library("Biostrings") in
    the vignette workflow)
- name: mzID
  role: Loads the peptide identifications from the mzIdentML file (Thermo_Hela_PRTC_selected.mzid)
    that populate the Proteins object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:d3d8809954443d640f130b2596ee950ed564ffd4b3f5fbcf58e5faad7e17bddb
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
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# protein-sequence-loading-from-fasta

## Summary

Load protein sequences from a fasta file into Pbase `Proteins` instances using Biostrings, then attach mzIdentML peptide identifications (mzID package) so peptides are mapped as coordinates along the proteins — the foundational data-assembly step for Pbase protein/peptide visualization.

## When to use

Use when protein sequences live in a fasta file (e.g. HUMAN_2015_02_selected.fasta from UniProt 2015_02) and peptide identifications from an mzIdentML file (e.g. Thermo_Hela_PRTC_selected.mzid) must be combined into Proteins objects for plotting.

## When NOT to use

- Peptide identifications are in a raw search-engine output format rather than mzIdentML — Pbase interoperates with identification results via the mzID or (devel) mzR packages, but the documented workflow here reads mzid files
- The goal is protein domain annotation rather than observed peptides — protein domains come from the EnsemblDb integration (Pbase-with-ensembldb vignette), not fasta+mzid loading
- The data is already a quantitative matrix (e.g. MSnSet) rather than sequence-level fasta/mzid inputs

## Inputs

- Protein fasta file (e.g. HUMAN_2015_02_selected.fasta, high scoring human proteins from UniProt release 2015_02)
- Peptide identifications in mzIdentML format (e.g. Thermo_Hela_PRTC_selected.mzid from LC-MSMS on a Thermo Orbitrap Q Exactive, searched with MSGF+)

## Outputs

- Pbase `Proteins` instances with protein sequences and peptide ranges mapped as coordinates along the proteins
- Per-protein peptide-to-protein coordinate mappings (ranges of interest such as experimentally observed peptides)

## How to apply

Read the protein sequences from the fasta file via Biostrings, e.g. `fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase")`; the example fasta is a restricted set of high scoring human proteins from UniProt release 2015_02. Load peptide identifications from the mzIdentML file with the mzID package (`idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package = "Pbase")`); in the source data the peptides originate from an LC-MSMS experiment on a Thermo Orbitrap Q Exactive, searched with MSGF+. Assemble the fasta sequences and mzID identifications into `Proteins` instances, the central data object of Pbase, so each protein links to a set of ranges of interest (e.g. experimentally observed peptides) that carry their own annotation. The peptides are mapped as coordinates along the proteins, which is what enables downstream plotting of peptide ranges aligned to protein sequences (e.g. proteins A4UGR9 and P00558).

## Related tools

- **Pbase** (Provides the `Proteins` central data object; sequences from a fasta file and peptides from an LC-MSMS experiment (mzid) are assembled into Proteins instances with peptides mapped as coordinates along proteins) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **Biostrings** (Reads the protein sequences from the fasta file (library("Biostrings") in the vignette workflow))
- **mzID** (Loads the peptide identifications from the mzIdentML file (Thermo_Hela_PRTC_selected.mzid) that populate the Proteins object)

## Examples

```
library("Biostrings"); library("mzID"); fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase"); idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package = "Pbase")
```

## Evaluation signals

- The resulting object is a valid Pbase `Proteins` instance containing protein sequences from the fasta file
- Each protein links to a set of ranges of interest (experimentally observed peptides) that carry their own annotation
- Peptide ranges align to the protein sequences as coordinates along the proteins
- Proteins of interest (e.g. A4UGR9 and P00558) are retrievable by identifier from the populated Proteins object

## Limitations

- The example fasta is a restricted set of high scoring human proteins from UniProt release 2015_02, not a full proteome
- Pbase is under heavy development and likely to considerably change in the near future (per the package README)

## Evidence

- [intro] Proteins objects are populated by protein sequences from a fasta file and peptides typically from an LC-MSMS experiment: "`Proteins` objects are populated by protein sequences stemming from a fasta file and the peptides typically originate from an LC-MSMS experiment."
- [intro] Peptides mapped as coordinates along proteins: "an optional set of peptides features that are mapped as coordinates along the proteins"
- [intro] fasta file path via system.file: "fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package = "Pbase")"
- [intro] mzid file path via system.file: "idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package = "Pbase")"
- [intro] Each protein links to ranges of interest with own annotation: "Each protein links to a set of ranges of interest, such as protein domains of experimentally observed peptides"
- [readme] Interoperability with mzID identification results: [evidence span withheld — anchor status: external_doc]
