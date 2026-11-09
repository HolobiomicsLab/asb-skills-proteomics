---
name: mzid-identification-parsing
description: Use when you have mzIdentML (.mzid) identification output from an LC-MSMS
  search (e.g. MSGF+ on Thermo Orbitrap Q Exactive data) and need to map identified
  peptides onto protein sequences in R.
when_to_use_negative:
- You need raw MS spectra handling rather than identification results — mzID parses
  identification files, while raw data would use mzR/MSnExp instances
- Input is a quantification feature table (e.g. an MSnSet) rather than an mzIdentML
  identification file, since no peptide-to-protein coordinate mapping from mzid is
  possible
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: mzID
  role: Parses the mzIdentML identification file (Thermo_Hela_PRTC_selected.mzid)
    to supply peptide identifications for populating Proteins instances
  repo: https://github.com/lgatto/Pbase
- name: Pbase
  role: Assembles mzID identifications and FASTA sequences into `Proteins` instances,
    mapping peptides as coordinates along proteins
  repo: https://github.com/lgatto/Pbase
- name: Biostrings
  role: Reads the protein sequences from the FASTA file used to populate the Proteins
    object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:652d87a927471fd43fc3bacd443957bc25770c28b6aa07e9da18389c0c0d7be7
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
  - mzID
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mzid-identification-parsing

## Summary

Parse peptide-spectrum identification results from mzIdentML (mzid) files with the Bioconductor mzID package and assemble them with FASTA protein sequences into Pbase `Proteins` instances, so peptides are mapped as coordinates along proteins for plotting.

## When to use

Use when you have mzIdentML (.mzid) identification output from an LC-MSMS search (e.g. MSGF+ on Thermo Orbitrap Q Exactive data) and need to map identified peptides onto protein sequences in R.

## When NOT to use

- You need raw MS spectra handling rather than identification results — mzID parses identification files, while raw data would use mzR/MSnExp instances
- Input is a quantification feature table (e.g. an MSnSet) rather than an mzIdentML identification file, since no peptide-to-protein coordinate mapping from mzid is possible

## Inputs

- mzIdentML (.mzid) peptide identification file, e.g. Thermo_Hela_PRTC_selected.mzid from Pbase extdata
- Protein sequence FASTA file, e.g. HUMAN_2015_02_selected.fasta (UniProt 2015_02 high-scoring human proteins)

## Outputs

- Pbase `Proteins` instances with protein sequences, annotation data, and identified peptides mapped as coordinate ranges along the proteins

## How to apply

Load the mzid file with the mzID package (`library("mzID")`) and the corresponding protein sequences with Biostrings from a FASTA file (e.g. Pbase's extdata HUMAN_2015_02_selected.fasta, a restricted set of high-scoring human proteins from UniProt release 2015_02). Pass both objects to Pbase's `Proteins` constructor, which links each protein sequence to its identified peptide ranges; the peptides originate from the LC-MSMS experiment recorded in the mzid and are mapped as coordinates along the proteins. Use the resulting `Proteins` instance as input to downstream visualization, such as plotting proteins A4UGR9 and P00558 (P00558 = tryptic-peptide standard protein) with their experimentally observed peptide ranges. The rationale is that `Proteins` is Pbase's central data object linking sequences to ranges of interest (domains or observed peptides) with their own annotation.

## Related tools

- **mzID** (Parses the mzIdentML identification file (Thermo_Hela_PRTC_selected.mzid) to supply peptide identifications for populating Proteins instances) — https://github.com/lgatto/Pbase
- **Pbase** (Assembles mzID identifications and FASTA sequences into `Proteins` instances, mapping peptides as coordinates along proteins) — https://github.com/lgatto/Pbase
- **Biostrings** (Reads the protein sequences from the FASTA file used to populate the Proteins object)

## Examples

```
library(mzID); idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package="Pbase"); ids <- mzID(idfile)
```

## Evaluation signals

- The `Proteins` object is populated with sequences from the FASTA file and peptide ranges from the mzid file; each protein links to a set of ranges of interest
- Plotted peptide ranges align to the protein sequences (e.g. A4UGR9 and P00558 tracks in Figure 6 panels)
- All identified peptides from the mzid are mapped as coordinates along the correct proteins
- mzID reads the file without error and `Proteins` construction completes (e.g. with knitr cache=TRUE chunk)

## Limitations

- Pbase was under heavy development and likely to considerably change; the README recommends checking `?Proteins` and the Pbase-data vignette
- The example is limited to a restricted set of high scoring human proteins from UniProt 2015_02 searched with MSGF+; larger or other-engine mzid files are not demonstrated here
- Only the mzID package path is shown in the vignette; (devel) mzR support for identification results is mentioned as possible interoperability

## Evidence

- [intro] Proteins objects are populated by protein sequences from a fasta file and peptides typically from an LC-MSMS experiment: "`Proteins` objects are populated by protein sequences stemming from a
fasta file and the peptides typically originate from an LC-MSMS
experiment."
- [intro] mzID library loading in vignette chunk: "{r psm, cache=TRUE}
library("mzID")"
- [intro] extdata file paths for fasta and mzid: "fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta",
                      package = "Pbase")"
- [intro] Peptides mapped as coordinates along proteins: "an optional set of
peptides features that are mapped as coordinates along the proteins"
- [readme] README interoperability with mzID and AAString: "The package allows to easily interact with `AAString` and
`AAStringSet` instances, protein databases such as UniProt (and
possibly biomaRt in the future) using protein identifiers,"
