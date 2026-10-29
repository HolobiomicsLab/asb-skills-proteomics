---
name: protein-sequence-conservation-quantification
description: Use when a set of protein sequence variants (e.g. from a BLAST search
  or a UniProt/Ensembl variant download) has been aligned with MUSCLE and per-position
  absolute and percent amino acid occurrence matrices are needed downstream of or
  alongside 3D structure conservation mapping.
when_to_use_negative:
- The analysis goal is SNP variant frequency/type annotation rather than amino acid
  occurrence/conservation (scop3D's -var mode produces a different matrix annotating
  the protein sequence with frequency, variants and type of variant).
- No aligned set of variant sequences exists and only a single reference sequence
  is available — per-position occurrence counts require multiple variants.
- The goal is already-solved downstream output such as the modified PDB files with
  B-values replaced by percent conservation or entropy; this skill covers only the
  per-position occurrence matrices stage.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: scop3D
  role: Pipeline that implements the per-position amino acid occurrence computation
    and writes the two occurrence matrices to the output directory
  repo: github.com/compomics/scop3d
- name: Biopython
  role: Python dependency used for reading the FASTA/multiple sequence alignment during
    sequence conservation analysis
- name: MUSCLE
  role: Aligns the sequence variants into the multiple sequence alignment from which
    per-position occurrences are tallied
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:db2a466046d9f01aeadc313ac349c22cce98c598662a60fc1a03495a4678ba47
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Biopython
  - MUSCLE
  - scop3D
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# protein-sequence-conservation-quantification

## Summary

Computes per-position amino acid occurrence statistics from a multiple sequence alignment of protein variants, producing two text-file matrices that annotate the consensus sequence with absolute and relative (percent) abundances of each amino acid, as implemented in scop3D.

## When to use

Use when a set of protein sequence variants (e.g. from a BLAST search or a UniProt/Ensembl variant download) has been aligned with MUSCLE and per-position absolute and percent amino acid occurrence matrices are needed downstream of or alongside 3D structure conservation mapping.

## When NOT to use

- The analysis goal is SNP variant frequency/type annotation rather than amino acid occurrence/conservation (scop3D's -var mode produces a different matrix annotating the protein sequence with frequency, variants and type of variant).
- No aligned set of variant sequences exists and only a single reference sequence is available — per-position occurrence counts require multiple variants.
- The goal is already-solved downstream output such as the modified PDB files with B-values replaced by percent conservation or entropy; this skill covers only the per-position occurrence matrices stage.

## Inputs

- Multiple sequence alignment of protein sequence variants (aligned FASTA produced by MUSCLE)
- User-defined conservation threshold (residue must be found more often than this threshold to be retained in the consensus)

## Outputs

- Text file matrix annotating the consensus sequence with the absolute abundance of each amino acid per position
- Text file matrix of relative amino acid abundances in percent per position
- Consensus sequence (consensus.fasta) used as BLAST input for structure retrieval

## How to apply

Read the multiple sequence alignment (produced by aligning variant sequences with MUSCLE, typically from a FASTA file of sequence variants) using Biopython. For each alignment position, tally the absolute count of every amino acid across all variant sequences; the consensus residue at a position is the most frequently found residue, and it must exceed the user-set conservation threshold to be retained. Build the first matrix annotating the consensus sequence with the absolute abundance of each amino acid per position, then convert these counts into relative abundances in percent for the second matrix. Write both matrices as two separate text files in the working output directory (scop3D bundles all input/output via the -w directory, e.g. ~/outputdir). These per-position occurrence statistics feed the downstream structure-annotation stage, where percent conservation or entropy is written into B-values of a PDB file.

## Related tools

- **scop3D** (Pipeline that implements the per-position amino acid occurrence computation and writes the two occurrence matrices to the output directory) — github.com/compomics/scop3d
- **Biopython** (Python dependency used for reading the FASTA/multiple sequence alignment during sequence conservation analysis)
- **MUSCLE** (Aligns the sequence variants into the multiple sequence alignment from which per-position occurrences are tallied)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta
```

## Evaluation signals

- Each matrix text file contains one entry per amino acid per alignment position.
- All values in the relative-abundance matrix lie in the range 0–100 percent.
- For each position, the percentages in the relative-abundance matrix sum to at most 100.
- The consensus residue at each position equals the most frequently found residue across variant sequences and exceeds the user-set conservation threshold.
- Both text files are present in the -w output directory alongside other scop3D outputs such as consensus.fasta.

## Limitations

- The consensus residue is only retained if it is found more often than the user-defined conservation threshold; positions below threshold lack a consensus designation.
- Sequence alignment quality depends on the upstream variant set (BLAST identity cutoff or UniProt/Ensembl variant download), so gapped or divergent alignments directly distort per-position occurrence counts.
- The percent abundance matrix alone does not convey information entropy — entropy is computed separately and reported in the second modified PDB file.

## Evidence
