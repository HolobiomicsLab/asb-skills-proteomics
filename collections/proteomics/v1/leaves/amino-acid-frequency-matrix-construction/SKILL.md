---
name: amino-acid-frequency-matrix-construction
description: Use when you have a MUSCLE multiple sequence alignment of protein variants
  and need per-position absolute and relative amino acid occurrence tables, or a consensus
  sequence with user-defined conservation thresholding.
when_to_use_negative:
- The goal is SNP-level variant reporting (frequency, variants, type of variant per
  position) rather than amino acid occurrence matrices — scop3D's SNP variants analysis
  produces a different matrix annotating the protein sequence with SNP frequency and
  type.
- You need conservation mapped onto a 3D structure (percentage conservation or information
  entropy in PDB B-values) — that is the downstream structure-annotation part, which
  consumes these matrices rather than producing them.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: scop3D
  role: Pipeline that constructs the consensus sequence and the absolute and relative
    amino acid occurrence matrices from the multiple sequence alignment and writes
    them as text files
  repo: github.com/compomics/scop3d
- name: MUSCLE
  role: Aligns the sequence variants to produce the multiple sequence alignment that
    the per-position tallies are computed from
- name: Biopython
  role: Python dependency used to read the aligned FASTA file for per-position amino
    acid counting
- name: WebLogo
  role: Creates a sequence logo from the same alignment after the matrices are built
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:ea580cd3a0f1d66e204c194c6f6a05cb30c23084999f4a693a861fe77dcef690
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Biopython
  - MUSCLE
  - WebLogo
  - scop3D
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# amino-acid-frequency-matrix-construction

## Summary

Builds two per-position matrices from a multiple sequence alignment: one annotating the consensus sequence with the absolute abundance of each amino acid per position, and one converting those counts into relative abundances in percent. This is the scop3D sequence-conservation reporting step that yields the text-file outputs backing the tool's 3D conservation visualization.

## When to use

Use when you have a MUSCLE multiple sequence alignment of protein variants and need per-position absolute and relative amino acid occurrence tables, or a consensus sequence with user-defined conservation thresholding.

## When NOT to use

- The goal is SNP-level variant reporting (frequency, variants, type of variant per position) rather than amino acid occurrence matrices — scop3D's SNP variants analysis produces a different matrix annotating the protein sequence with SNP frequency and type.
- You need conservation mapped onto a 3D structure (percentage conservation or information entropy in PDB B-values) — that is the downstream structure-annotation part, which consumes these matrices rather than producing them.

## Inputs

- FASTA file containing all sequence variants of interest (or a MUSCLE multiple sequence alignment of those variants)
- User-defined conservation threshold (the consensus residue must be found more than this threshold)

## Outputs

- Text file: matrix annotating the consensus sequence with the absolute abundance of each amino acid per alignment position
- Text file: matrix of relative amino acid abundances per position in percent
- consensus.fasta in the output directory (usable as BLAST input against the PDB)

## How to apply

Read the multiple sequence alignment (produced by aligning the variant sequences with MUSCLE) from the FASTA file using Biopython. For each alignment position, tally the absolute count of every amino acid across all variant sequences; construct the consensus sequence by retaining the most frequently found residue per position, applying the user-set conservation threshold so the consensus residue must be found more often than that threshold. Build the first matrix annotating the consensus sequence with the absolute abundance of each amino acid per position, then convert these numbers into relative abundances in percent for the second matrix. Write both matrices to separate text files in the working output directory; a sequence logo can subsequently be created with WebLogo.

## Related tools

- **scop3D** (Pipeline that constructs the consensus sequence and the absolute and relative amino acid occurrence matrices from the multiple sequence alignment and writes them as text files) — github.com/compomics/scop3d
- **MUSCLE** (Aligns the sequence variants to produce the multiple sequence alignment that the per-position tallies are computed from)
- **Biopython** (Python dependency used to read the aligned FASTA file for per-position amino acid counting)
- **WebLogo** (Creates a sequence logo from the same alignment after the matrices are built)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta
```

## Evaluation signals

- Each matrix text file contains one entry per amino acid per alignment position.
- Relative-abundance matrix values fall in the range 0–100 percent.
- Each position's relative percentages sum to at most 100.
- The consensus residue at each position matches the most frequently found residue across variant sequences and exceeds the user-defined conservation threshold.
- A consensus.fasta is present in the output directory and can be used as BLAST input.

## Limitations

- The consensus residue is only retained if it is found more than the user-defined conservation threshold; positions below the threshold may lack a consensus residue.
- All files are read from and written to the directory given with -w; subsequent runs search for input files in that same directory, so path handling is constrained.
- Absolute counts reflect only the supplied variant sequences (or BLAST-retrieved sequences subject to the identity cutoff), so the matrices are bounded by the diversity of that input set.

## Evidence

- [readme] Two matrices are created. The first matrix annotates the consensus sequence with the absolute abundance of each amino acid per position. In the second matrix, these numbers are converted into relative abundances in percent.: "Two matrices are created. The first matrix annotates the consensus sequence with the absolute abundance of each amino acid per position."
- [abstract] text files with absolute and relative amino acid occurrences for each position are also provided, along with snapshots of the protein from six distinct directions in space.: "text files with absolute and relative amino acid occurrences for each position are also provided"
- [methods] the consensus sequence is constructed by retaining the most frequently found residue across all variant sequences for each position. This residue should be found more than the conservation threshold which is set be the user.: "the consensus sequence is constructed by retaining the most frequently found residue across all variant sequences for each position. This residue should be found more than the conservation threshold"
- [readme] These variants are aligned with MUSCLE. From the resulting multiple sequence alignment, the consensus sequence is constructed: "These variants are aligned with MUSCLE. From the resulting multiple sequence alignment, the consensus sequence is constructed"
