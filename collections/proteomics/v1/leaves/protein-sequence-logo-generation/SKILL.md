---
name: protein-sequence-logo-generation
description: Use when a FASTA file of protein sequence variants (or a MUSCLE alignment)
  has been analyzed for conservation and a visual, per-position residue-frequency
  representation is required.
when_to_use_negative:
- Input is raw sequence variant data that has not yet been aligned — run the MUSCLE
  alignment step first
- The goal is SNP-variant frequency annotation on a protein sequence rather than conservation
  visualization per position
- No alignment or per-position frequency data exists, e.g. only a single sequence
  with no variants to compare
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: Scop3D
  role: 'Orchestrates the sequence conservation analysis: aligns variants, builds
    the consensus, computes per-position absolute and relative amino acid abundances
    that underlie the logo'
  repo: github.com/compomics/scop3d
- name: MUSCLE
  role: Aligns the sequence variants; the resulting multiple sequence alignment is
    the input from which the logo is rendered
- name: WebLogo
  role: Creates the sequence logo as the final step of the scop3D sequence conservation
    analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:19fed58d1f14b3f3878e5e5b20b90a34f33e6dd55a7eb78f9742d8a50aaa1e6a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MUSCLE
  - Scop3D
  - WebLogo
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# protein-sequence-logo-generation

## Summary

Generate a sequence logo that visualizes per-position amino acid conservation from a multiple sequence alignment, as the final step of the scop3D workflow. Stack heights reflect residue information content computed from the same alignment frequencies that underlie the consensus sequence.

## When to use

Use when a FASTA file of protein sequence variants (or a MUSCLE alignment) has been analyzed for conservation and a visual, per-position residue-frequency representation is required.

## When NOT to use

- Input is raw sequence variant data that has not yet been aligned — run the MUSCLE alignment step first
- The goal is SNP-variant frequency annotation on a protein sequence rather than conservation visualization per position
- No alignment or per-position frequency data exists, e.g. only a single sequence with no variants to compare

## Inputs

- FASTA file containing all sequence variants of interest (e.g. sequences.fasta)
- MUSCLE multiple sequence alignment of the variants
- Per-position absolute and relative amino acid occurrence matrices computed by scop3D

## Outputs

- Sequence logo image file (e.g. sequence_logo.png) where stack height reflects residue conservation per position

## How to apply

Start from the MUSCLE multiple sequence alignment of all sequence variants of interest — the same alignment from which scop3D builds the consensus sequence by retaining the most frequently found residue per position (above the user-set conservation threshold). The per-position absolute and relative amino acid abundances computed in the two scop3D matrices underlie the logo, so the logo must depict the most frequent residues at each position. Render the logo with WebLogo (listed with MUSCLE and EMBOSS in scop3D's system requirements). Validate that the output logo image exists, is a readable image file, and that its per-position letters match the most frequent residues in the alignment.

## Related tools

- **Scop3D** (Orchestrates the sequence conservation analysis: aligns variants, builds the consensus, computes per-position absolute and relative amino acid abundances that underlie the logo) — github.com/compomics/scop3d
- **MUSCLE** (Aligns the sequence variants; the resulting multiple sequence alignment is the input from which the logo is rendered)
- **WebLogo** (Creates the sequence logo as the final step of the scop3D sequence conservation analysis)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta
```

## Evaluation signals

- The sequence logo image file (e.g. sequence_logo.png) exists in the output directory and is a valid, readable image
- Per-position residue letters depicted in the logo correspond to the most frequent residues in the MUSCLE alignment
- Stack heights at each position are consistent with the relative abundance matrix (percent per position) computed by scop3D
- Consensus residues retained in consensus.fasta agree with the dominant letters in the logo above the user-set conservation threshold
- WebLogo, MUSCLE, and EMBOSS are listed as installed dependencies (scop3D system requirements) so the rendering step can run

## Limitations

- The logo is only as informative as the alignment: consensus residues must exceed the user-defined conservation threshold, otherwise per-position frequencies may reflect weak conservation
- The logo visualizes the conservation analysis only; the parallel SNP variants analysis path produces frequency/type matrices and colored sequences instead
- scop3D requires Python >= 2.7, Perl >= 5, Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss — missing dependencies will prevent logo generation

## Evidence
