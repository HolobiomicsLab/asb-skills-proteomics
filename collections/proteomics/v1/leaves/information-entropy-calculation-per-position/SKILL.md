---
name: information-entropy-calculation-per-position
description: Use when a multiple sequence alignment of protein variants exists (or
  can be built with MUSCLE) and per-position conservation diversity must be quantified
  as information entropy and mapped onto a 3D structure via B-values.
when_to_use_negative:
- The goal is SNP frequency/type mapping rather than conservation entropy — use the
  SNP variants analysis mode ('-var'), which replaces B-values with SNP frequency
  and SNP type instead
- The input is raw unassembled variant DNA or a single unaligned sequence with no
  multiple sequence alignment possible — entropy per position requires aligned variant
  sequences
- Only a sequence-level logo/consensus is needed and no 3D structure mapping is required
  — the structure annotation step is unnecessary
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_2814
tools:
- name: scop3D
  role: Computes per-position sequence conservation and information entropy and replaces
    B-values in the output PDB files; invoked with '-ent' for entropy mode
  repo: https://github.com/compomics/scop3d
- name: MUSCLE
  role: Aligns the FASTA sequence variants to produce the multiple sequence alignment
    from which per-position occurrence counts and entropy are derived
- name: BLAST
  role: Retrieves sequence variants with an identity cutoff and matches consensus.fasta
    against the PDB to select the structure for annotation
- name: WebLogo
  role: Creates the sequence logo visualizing per-position amino-acid abundances alongside
    the entropy calculation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:6a45bc320a1682a01add6e64fc85d997e6161ccb6bedcfb009c6edabbb41f453
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/compomics/scop3d
  tools:
  - BLAST
  - MUSCLE
  - Scop3D
  - WebLogo
  - scop3D
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_2814
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# information-entropy-calculation-per-position

## Summary

Computes the information entropy of amino-acid occurrence per position across aligned sequence variants, and writes it into the B-value column of a PDB structure file so that conservation entropy can be visualized directly on the protein 3D structure.

## When to use

Use when a multiple sequence alignment of protein variants exists (or can be built with MUSCLE) and per-position conservation diversity must be quantified as information entropy and mapped onto a 3D structure via B-values.

## When NOT to use

- The goal is SNP frequency/type mapping rather than conservation entropy — use the SNP variants analysis mode ('-var'), which replaces B-values with SNP frequency and SNP type instead
- The input is raw unassembled variant DNA or a single unaligned sequence with no multiple sequence alignment possible — entropy per position requires aligned variant sequences
- Only a sequence-level logo/consensus is needed and no 3D structure mapping is required — the structure annotation step is unnecessary

## Inputs

- FASTA file of sequence variant sequences (e.g. sequences.fasta), or a Uniprot ID / DNA sequence with identity cutoff for BLAST-based variant retrieval
- User-set identity cutoff (-t) and conservation threshold
- PDB file (from the PDB or user-supplied) or a PDB ID chosen after BLASTing consensus.fasta

## Outputs

- Two modified PDB files with B-values replaced by percent variation (first) and information entropy per position (second)
- Text files with absolute and relative amino-acid occurrences per position
- consensus.fasta in the output directory
- WebLogo sequence logo and snapshots of the protein from six directions in space

## How to apply

Start from a FASTA file of sequence variants (or retrieve variants by UniprotID / BLAST with a user-set identity cutoff via -t) and align them with MUSCLE. From the multiple sequence alignment, build the absolute and relative amino-acid occurrence matrices per position and construct a consensus retaining the most frequent residue only when its abundance exceeds the user-defined conservation threshold. Run scop3D in entropy mode ('./scop3d sequence -ent -w ~/outputdir -f sequences.fasta') to compute the information entropy per position; the sequence step also writes consensus.fasta and a WebLogo sequence logo to the output directory. Then run the structure step ('./scop3d structure -ent -w ~/outputdir -i <PDB_ID>', optionally restricting chains with -c A,C) so that in the second output PDB file the B-values are replaced by the entropy factor per position, while the first output PDB carries percent variation (100 minus percentage conservation). Correctness is judged by the presence of exactly two modified PDB files whose B-factor columns carry percent-variation and entropy values for every annotated residue position.

## Related tools

- **scop3D** (Computes per-position sequence conservation and information entropy and replaces B-values in the output PDB files; invoked with '-ent' for entropy mode) — https://github.com/compomics/scop3d
- **MUSCLE** (Aligns the FASTA sequence variants to produce the multiple sequence alignment from which per-position occurrence counts and entropy are derived)
- **BLAST** (Retrieves sequence variants with an identity cutoff and matches consensus.fasta against the PDB to select the structure for annotation)
- **WebLogo** (Creates the sequence logo visualizing per-position amino-acid abundances alongside the entropy calculation)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta && ./scop3d blast ~/outputdir/consensus.fasta && ./scop3d structure -ent -w ~/outputdir -i 4F15
```

## Evaluation signals

- Exactly two modified PDB files appear in the output directory after the structure step
- In the second output PDB, the B-factor column contains the entropy factor for every annotated residue position
- In the first output PDB, the B-factor column contains percent variation (100 minus percentage conservation) per position
- consensus.fasta is written to the output directory and reflects the most frequently found residue per position retained only above the user-set conservation threshold
- Absolute and relative amino-acid occurrence text files accompany the run and are consistent with the alignment

## Limitations

- Residues whose most frequent amino acid does not exceed the user-defined conservation threshold are not retained in the consensus
- Requires a matching or homologous PDB structure; if none exists, a BLAST of the consensus against the PDB is used, which may not return a suitable structure
- Output depends on whether conservation ('-ent') or SNP variants ('-var') analysis was run first — mixing modes will change what the B-values represent

## Evidence
