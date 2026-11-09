---
name: sequence-conservation-analysis
description: Use when you have a FASTA file of protein sequence variants (or a UniprotID)
  and need to quantify per-position conservation/entropy and visualize it on a 3D
  protein structure from the PDB.
when_to_use_negative:
- The goal is SNP variant analysis (frequency and SNP type per position) rather than
  conservation/entropy — scop3D's -var mode is the appropriate path instead
- No protein structure or homologous PDB structure is needed and only a sequence logo
  or consensus is required — MUSCLE/WebLogo alone suffices
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0166
tools:
- name: scop3D
  role: Main tool that orchestrates the conservation analysis and maps conservation/entropy
    onto the 3D structure
  repo: https://github.com/compomics/scop3d
- name: MUSCLE
  role: Aligns the sequence variants into a multiple sequence alignment from which
    the consensus is built
- name: BLAST
  role: Retrieves variants against NCBI with an identity cutoff and finds a matching
    PDB structure for the consensus
- name: WebLogo
  role: Creates the sequence logo from the occurrence matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:088ae527ef4fb49a9653a3ef48f174066b4c363573f1b1577faf57acf603900d
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0166
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# sequence-conservation-analysis

## Summary

Performs per-position sequence conservation analysis on a set of protein sequence variants using scop3D: aligning variants with MUSCLE, building a consensus and occurrence matrices, and mapping conservation or entropy values back onto a PDB structure as B-values.

## When to use

Use when you have a FASTA file of protein sequence variants (or a UniprotID) and need to quantify per-position conservation/entropy and visualize it on a 3D protein structure from the PDB.

## When NOT to use

- The goal is SNP variant analysis (frequency and SNP type per position) rather than conservation/entropy — scop3D's -var mode is the appropriate path instead
- No protein structure or homologous PDB structure is needed and only a sequence logo or consensus is required — MUSCLE/WebLogo alone suffices

## Inputs

- FASTA file of protein sequence variants (sequences.fasta)
- UniprotID with identity cutoff (alternative variant source)
- PDB file or PDB ID for the structure annotation step
- User-defined conservation threshold and identity cutoff parameters

## Outputs

- Two modified PDB files with B-values replaced by percent variation and entropy per position
- consensus.fasta written to the output directory
- Text files with absolute and relative amino acid occurrences per position
- Sequence logo (WebLogo)
- Snapshots of the protein from six distinct directions in space

## How to apply

Start from a FASTA file containing all sequence variants of interest (or retrieve variants via a UniprotID with BLAST and a user-set identity cutoff, e.g. -t 50). Align the variants with MUSCLE and construct the consensus by retaining the most frequently found residue per position, keeping it only when its abundance exceeds the user-defined conservation threshold. Build the absolute and relative amino-acid occurrence matrices and generate a sequence logo with WebLogo; the run also writes consensus.fasta to the output directory. Optionally BLAST the consensus against the PDB to pick a matching structure (e.g. 4F15), then run the structure step so that in one output PDB the B-values are replaced by percent variation (100 minus percentage conservation) and in the other by the entropy factor per position.

## Related tools

- **scop3D** (Main tool that orchestrates the conservation analysis and maps conservation/entropy onto the 3D structure) — https://github.com/compomics/scop3d
- **MUSCLE** (Aligns the sequence variants into a multiple sequence alignment from which the consensus is built)
- **BLAST** (Retrieves variants against NCBI with an identity cutoff and finds a matching PDB structure for the consensus)
- **WebLogo** (Creates the sequence logo from the occurrence matrices)

## Examples

```
./scop3d sequence -ent -w ~/outputdir -f sequences.fasta; ./scop3d blast ~/outputdir/consensus.fasta; ./scop3d structure -ent -w ~/outputdir -i 4F15
```

## Evaluation signals

- Exactly two modified PDB files are produced in the output directory for a conservation analysis run
- B-factor columns of the output PDBs carry percent-variation (100 minus percent conservation) and entropy values for every annotated residue position
- consensus.fasta exists in the output directory and contains the most frequently found residue per position where abundance exceeded the conservation threshold
- Occurrence matrices report absolute abundances per position and the same numbers converted to relative percentages
- A sequence logo file and six directional structure snapshots are present among the outputs

## Limitations

- The consensus residue is only retained when found more than the user-set conservation threshold, so positions below threshold may be unannotated
- Structure annotation requires a PDB file or a BLAST match of the consensus against the PDB; if no homologous structure exists, mapping cannot be done
- Standalone tool requires Python >= 2.7, Perl >= 5, Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss

## Evidence
