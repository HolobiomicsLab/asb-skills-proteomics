---
name: multiple-sequence-alignment-residue-tallying
description: Use when a multiple sequence alignment (e.g. a MUSCLE-aligned FASTA of
  protein variants) must be converted into per-position amino acid occurrence counts
  and percentages to build a consensus or conservation report.
when_to_use_negative:
- 'The input is a DNA/SNP-variant analysis scenario: scop3D then produces a matrix
  annotating the protein sequence with SNP frequency, variants and variant type instead
  of amino acid occurrence matrices'
- Only a single unaligned sequence is available and no multiple sequence alignment
  has been produced with MUSCLE yet
- The goal is structural annotation (PDB B-value replacement with percent conservation
  or entropy) rather than per-position amino acid composition reporting
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: scop3D
  role: Pipeline that aligns variants, builds the consensus, and writes the absolute
    and relative amino acid occurrence matrices as text files to the output directory
  repo: https://github.com/compomics/scop3d
- name: Biopython
  role: Python library used to read the aligned FASTA and iterate over alignment positions
    when tallying amino acid occurrences
- name: MUSCLE
  role: Aligns the sequence variant FASTA into the multiple sequence alignment that
    is the input for residue tallying
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:9bc19a3181372a2dc31d0caece022d1b1224b7e8622f81ab070f60183acff383
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/scop3d
  tools:
  - Biopython
  - MUSCLE
  - scop3D
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# multiple-sequence-alignment-residue-tallying

## Summary

Reconstructs scop3D's per-position amino acid occurrence computation: after aligning protein sequence variants with MUSCLE, tally absolute and relative abundances of every amino acid at each alignment position and write them as two separate text matrices. This underpins consensus-sequence construction and downstream 3D conservation mapping.

## When to use

Use when a multiple sequence alignment (e.g. a MUSCLE-aligned FASTA of protein variants) must be converted into per-position amino acid occurrence counts and percentages to build a consensus or conservation report.

## When NOT to use

- The input is a DNA/SNP-variant analysis scenario: scop3D then produces a matrix annotating the protein sequence with SNP frequency, variants and variant type instead of amino acid occurrence matrices
- Only a single unaligned sequence is available and no multiple sequence alignment has been produced with MUSCLE yet
- The goal is structural annotation (PDB B-value replacement with percent conservation or entropy) rather than per-position amino acid composition reporting

## Inputs

- Aligned FASTA file containing the multiple sequence alignment of protein sequence variants (produced with MUSCLE)
- User-defined conservation threshold (a residue must be found more often than this threshold to be retained in the consensus)
- Working output directory path (e.g. ~/outputdir)

## Outputs

- Text file with the first matrix annotating the consensus sequence with the absolute abundance of each amino acid per alignment position
- Text file with the second matrix containing the relative amino acid abundances in percent per position

## How to apply

Read the aligned FASTA produced by MUSCLE into Biopython and iterate over every alignment column. For each position, tally the absolute count of every amino acid across all variant sequences; the consensus residue is the most frequently found residue, retained only if it exceeds the user-defined conservation threshold. Build a first matrix annotating the consensus sequence with the absolute abundance of each amino acid per position, then convert those counts into relative abundances in percent for a second matrix. Write both matrices to two separate text files in the working output directory (set with scop3D's -w flag).

## Related tools

- **scop3D** (Pipeline that aligns variants, builds the consensus, and writes the absolute and relative amino acid occurrence matrices as text files to the output directory) — https://github.com/compomics/scop3d
- **Biopython** (Python library used to read the aligned FASTA and iterate over alignment positions when tallying amino acid occurrences)
- **MUSCLE** (Aligns the sequence variant FASTA into the multiple sequence alignment that is the input for residue tallying)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta
```

## Evaluation signals

- Each matrix text file contains one entry per amino acid per alignment position
- Relative-abundance matrix values fall in the 0-100 percent range
- For each alignment position, the relative percentages sum to at most 100 percent
- The consensus residue at each position matches the most frequently found residue across all variant sequences and exceeds the user-set conservation threshold
- The two text files appear in the directory specified with the -w flag alongside the other scop3D outputs (consensus.fasta, modified PDB files, figures)

## Limitations

- The consensus residue is only retained if it is found more often than the user-defined conservation threshold; positions below the threshold are not assigned a consensus residue
- Matrix quality depends on the MUSCLE alignment; scop3D does not realign or curate the input alignment
- Requires Python >= 2.7, Perl >= 5, Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss in the standalone installation

## Evidence
