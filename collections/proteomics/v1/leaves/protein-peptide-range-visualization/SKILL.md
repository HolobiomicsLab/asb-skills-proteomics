---
name: protein-peptide-range-visualization
description: Use when identified peptides from an LC-MSMS mzIdentML file must be mapped
  as coordinate ranges onto protein sequences from a fasta file and visualized per
  protein in R.
when_to_use_negative:
- Input is already a quantitative feature/peptide table without mzIdentML identification
  files — Pbase `Proteins` objects are populated from fasta sequences and mzID identifications,
  not feature tables.
- The goal is raw MS spectrum plotting (e.g. chromatograms or spectra) rather than
  peptide positions along protein sequences.
- Protein identifiers are absent from the fasta/mzid pair, so peptides cannot be linked
  to protein sequences.
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pbase
  role: 'Central package: builds `Proteins` instances linking protein sequences to
    peptide ranges of interest and provides the protein plotting method used for the
    Figure 6 panels'
  repo: http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- name: Biostrings
  role: Reads the HUMAN_2015_02_selected.fasta protein sequences feeding the `Proteins`
    objects
- name: mzID
  role: Parses the Thermo_Hela_PRTC_selected.mzid mzIdentML identifications that supply
    the observed peptide coordinates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201400392
    title: Visualization of proteomics data using R and Bioconductor
  source_content_hash: sha256:c8d04e8951c2fb23bbe4946ced4454a58200382ea52bbe666fcd59f262fa5383
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
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# protein-peptide-range-visualization

## Summary

Reproduces the Proteomics (Gatto et al., 2015) Figure 6 panels by building Pbase `Proteins` instances from the HUMAN_2015_02_selected.fasta sequences (UniProt release 2015_02) and Thermo_Hela_PRTC_selected.mzid peptide identifications, then plotting identified peptides as coordinate ranges along proteins A4UGR9 and P00558.

## When to use

Use when identified peptides from an LC-MSMS mzIdentML file must be mapped as coordinate ranges onto protein sequences from a fasta file and visualized per protein in R.

## When NOT to use

- Input is already a quantitative feature/peptide table without mzIdentML identification files — Pbase `Proteins` objects are populated from fasta sequences and mzID identifications, not feature tables.
- The goal is raw MS spectrum plotting (e.g. chromatograms or spectra) rather than peptide positions along protein sequences.
- Protein identifiers are absent from the fasta/mzid pair, so peptides cannot be linked to protein sequences.

## Inputs

- HUMAN_2015_02_selected.fasta (protein sequences from Pbase extdata, UniProt release 2015_02)
- Thermo_Hela_PRTC_selected.mzid (mzIdentML peptide identifications from an LC-MSMS experiment, MSGF+ search of Thermo Orbitrap Q Exactive data)

## Outputs

- Proteins instances with peptides mapped as coordinate ranges along proteins
- figure6_panels.png rendering the Figure 6 panels (A4UGR9 and P00558 protein/peptide tracks, plus P00558 coordinate view with ENSEMBL gene model)

## How to apply

Read the protein sequences from the Pbase extdata fasta (HUMAN_2015_02_selected.fasta, a restricted set of high-scoring human proteins from UniProt 2015_02) using Biostrings, and load peptide identifications from Thermo_Hela_PRTC_selected.mzid with the mzID package. Assemble both into `Proteins` instances with Pbase, so each protein links to ranges of interest — here experimentally observed peptides — mapped as coordinates along the protein sequence. Plot protein A4UGR9 with its observed peptide ranges using Pbase's protein plotting method, then repeat for P00558 (P00558 with the ENSEMBL gene-model coordinate overlay completes the Figure 6 panels). Correctness is judged by whether peptide ranges align to the protein sequences and whether all three panels render with the correct proteins.

## Related tools

- **Pbase** (Central package: builds `Proteins` instances linking protein sequences to peptide ranges of interest and provides the protein plotting method used for the Figure 6 panels) — http://bioconductor.org/packages/devel/bioc/html/Pbase.html
- **Biostrings** (Reads the HUMAN_2015_02_selected.fasta protein sequences feeding the `Proteins` objects)
- **mzID** (Parses the Thermo_Hela_PRTC_selected.mzid mzIdentML identifications that supply the observed peptide coordinates)

## Examples

```
fafile <- system.file("extdata/HUMAN_2015_02_selected.fasta", package="Pbase"); idfile <- system.file("extdata/Thermo_Hela_PRTC_selected.mzid", package="Pbase")
```

## Evaluation signals

- All three Figure 6 panels render with the correct proteins (A4UGR9 and P00558).
- Peptide ranges align to the protein sequences (peptide start/end coordinates fall within protein length).
- `Proteins` objects contain sequences from HUMAN_2015_02_selected.fasta and identifications from Thermo_Hela_PRTC_selected.mzid.
- The P00558 panel displays the protein sequence with the overlaid ENSEMBL gene model.
- Panels are saved as figure6_panels.png without plotting errors.

## Limitations

- Pbase is under heavy development and its API is likely to change considerably (per the README), so code may need adaptation against newer Bioconductor versions.
- The demonstration data is a restricted, high-scoring subset (human UniProt 2015_02 proteins, PRTC HeLa mzid), not a full database; scale-up to complete fasta/mzid files may be slower or expose identifier-matching edge cases.
- Genomic coordinate views (ENSEMBL gene model overlay) depend on Ensembldb/mapping vignette support; protein-domain annotation is only available through Ensembldb integration.

## Evidence

- [intro] Proteins objects are populated by protein sequences stemming from a fasta file and the peptides typically originate from an LC-MSMS experiment.: "`Proteins` objects are populated by protein sequences stemming from a
fasta file and the peptides typically originate from an LC-MSMS
experiment."
- [intro] Each protein links to ranges of interest such as experimentally observed peptides.: "Each protein links to a set of ranges of interest, such as
protein domains of experimentally observed peptides"
- [intro] Peptides are mapped as coordinates along the proteins.: "an optional set of
peptides features that are mapped as coordinates along the proteins"
- [readme] Pbase interoperates with AAString/AAStringSet and identification results from mzID.: "The package allows to easily interact with `AAString` and
`AAStringSet` instances, protein databases such as UniProt (and
possibly biomaRt in the future) using protein identifiers,"
- [readme] Protein domain annotation and genome mapping rely on Ensembldb/mapping vignettes.: [evidence span withheld — anchor status: external_doc]
