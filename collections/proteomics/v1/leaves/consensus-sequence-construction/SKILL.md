---
name: consensus-sequence-construction
description: Use when you have a MUSCLE multiple sequence alignment of variant sequences
  (or BLAST-retrieved homologs with an applied identity cutoff) and need a per-position
  consensus protein sequence to drive conservation scoring or PDB structure lookup.
when_to_use_negative:
- You are running scop3D's SNP variants analysis based on UniProt/Ensembl data where
  the deliverable is per-position SNP frequency and SNP type rather than a residue-frequency
  consensus for conservation mapping
- The input is already a consensus sequence or a single sequence — the skill requires
  a set of aligned variant sequences to tally per-position residue abundances
- You need a nucleotide-level consensus with codon/indel handling; the described pipeline
  translates and aligns to obtain a protein consensus and does not detail codon-aware
  consensus rules
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: scop3D
  role: Pipeline that performs the consensus construction step and writes consensus.fasta
    plus the absolute and relative occurrence matrices to the -w output directory
  repo: github.com/compomics/scop3d
- name: MUSCLE
  role: Aligns the variant sequences; the resulting multiple sequence alignment is
    the direct input for per-position consensus residue tallying
- name: Biopython
  role: Used to read the aligned FASTA file of the MUSCLE multiple sequence alignment
    when reconstructing per-position amino-acid occurrences
- name: BLAST
  role: Consumes the produced consensus.fasta in the intermediate BLAST step against
    the NCBI database (with identity cutoff) or the PDB to retrieve homologous sequences
    or a matching structure
- name: WebLogo
  role: Creates the sequence logo from the per-position amino-acid occurrence information
    computed alongside the consensus
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:06b291d810a9b23d8f90baf480361cd59a39aeecce46a24c553436db206b9e19
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BLAST
  - Bioperl
  - Biopython
  - MUSCLE
  - Scop3D
  - WebLogo
  - scop3D
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# consensus-sequence-construction

## Summary

Constructs a consensus protein sequence from a multiple sequence alignment of protein variants by retaining, at each alignment position, the most frequently found residue, provided its abundance exceeds a user-defined conservation threshold. In scop3D this step also writes consensus.fasta, which serves as the BLAST input for downstream structure annotation.

## When to use

Use when you have a MUSCLE multiple sequence alignment of variant sequences (or BLAST-retrieved homologs with an applied identity cutoff) and need a per-position consensus protein sequence to drive conservation scoring or PDB structure lookup.

## When NOT to use

- You are running scop3D's SNP variants analysis based on UniProt/Ensembl data where the deliverable is per-position SNP frequency and SNP type rather than a residue-frequency consensus for conservation mapping
- The input is already a consensus sequence or a single sequence — the skill requires a set of aligned variant sequences to tally per-position residue abundances
- You need a nucleotide-level consensus with codon/indel handling; the described pipeline translates and aligns to obtain a protein consensus and does not detail codon-aware consensus rules

## Inputs

- Multiple sequence alignment of variant sequences produced by MUSCLE (aligned FASTA file)
- User-set conservation threshold for consensus residue retention
- Identity cutoff (e.g. 50% or 70%) applied when sequences are BLAST-retrieved from NCBI rather than supplied directly

## Outputs

- consensus.fasta — protein consensus sequence written to the scop3d -w output directory, usable as BLAST input
- Text file with absolute amino-acid occurrence matrix annotating the consensus sequence per position
- Text file with relative (percent, 0–100) amino-acid occurrence matrix per position

## How to apply

Align the collected variant sequences with MUSCLE, then read the aligned FASTA and, for each alignment position, retain the most frequently found residue across all variant sequences as the consensus residue. Apply the user-set conservation threshold: the winning residue must be found more than the threshold (e.g. the user-defined cutoff set via scop3d options) or the position is not fixed to that residue. In parallel, build the absolute amino-acid occurrence matrix and its relative-abundance (percent) counterpart that annotate the consensus sequence per position. Write the consensus protein sequence to consensus.fasta in the output directory specified with the scop3d -w option; this file then feeds the intermediate BLAST step (e.g. './scop3d blast ~/outputdir/consensus.fasta') against the NCBI or PDB databases to retrieve homologous sequences or a matching structure such as PDB 4F15.

## Related tools

- **scop3D** (Pipeline that performs the consensus construction step and writes consensus.fasta plus the absolute and relative occurrence matrices to the -w output directory) — github.com/compomics/scop3d
- **MUSCLE** (Aligns the variant sequences; the resulting multiple sequence alignment is the direct input for per-position consensus residue tallying)
- **Biopython** (Used to read the aligned FASTA file of the MUSCLE multiple sequence alignment when reconstructing per-position amino-acid occurrences)
- **BLAST** (Consumes the produced consensus.fasta in the intermediate BLAST step against the NCBI database (with identity cutoff) or the PDB to retrieve homologous sequences or a matching structure)
- **WebLogo** (Creates the sequence logo from the per-position amino-acid occurrence information computed alongside the consensus)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta
```

## Evaluation signals

- consensus.fasta exists in the scop3d -w output directory and contains a protein sequence suitable as BLAST input for the subsequent 'scop3d structure' step
- Every consensus residue at each position is the most frequently found residue across all variant sequences and its abundance exceeds the user-set conservation threshold
- The absolute occurrence matrix contains one count per amino acid per alignment position, and the relative matrix values lie in 0–100 percent with per-position percentages summing to at most 100
- The consensus sequence successfully BLASTs against the PDB (e.g. recovering a matching structure such as 4F15) or NCBI with the applied identity cutoff
- A sequence logo is generated with WebLogo from the same occurrence data, indicating the per-position tallies were computed over the full alignment

## Limitations

- A consensus residue is retained only if it is found more than the user-defined conservation threshold; positions below threshold are not fixed, so low-conservation positions may yield incomplete or degenerate consensus columns
- When sequences are not supplied directly, the set to be aligned depends on the BLAST identity cutoff applied against the NCBI database (e.g. 50% or 70%), so the consensus is sensitive to this cutoff choice
- The pipeline requires Python >= 2.7, Perl >= 5 with Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss installed

## Evidence

- [readme] From the resulting multiple sequence alignment, the consensus sequence is constructed by retaining the most frequently found residue across all variant sequences for each position.: "From the resulting multiple sequence alignment, the consensus sequence is constructed by retaining the most frequently found residue across all variant sequences for each position."
- [readme] This residue should be found more than the conservation threshold which is set be the user.: "This residue should be found more than the conservation threshold which is set be the user."
- [readme] the sequence will be blasted and cutoff value applied: "the sequence will be blasted and cutoff value applied"
- [readme] Two matrices are created. The first matrix annotates the consensus sequence with the absolute abundance of each amino acid per position.: "Two matrices are created. The first matrix annotates the consensus sequence with the absolute abundance of each amino acid per position."
- [other] Amongst result files in the outputdir, a consensus.fasta is created, that can be used as a BLAST input: "Amongst result files in the outputdir, a consensus.fasta is created, that can be used as a BLAST input"
