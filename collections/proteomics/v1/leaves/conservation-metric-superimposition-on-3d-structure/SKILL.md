---
name: conservation-metric-superimposition-on-3d-structure
description: Use when you have a multiple sequence alignment (or SNP matrix) with
  per-position conservation/entropy or SNP frequency/type values and a PDB structure
  on which to visualize them.
when_to_use_negative:
- No protein 3D structure is available or obtainable — the intermediate BLAST step
  against the PDB must first retrieve a homologous structure before annotation can
  run.
- You need visualization of absolute/relative amino acid occurrences rather than per-position
  mapping onto structure — those are delivered as text files and WebLogo sequence
  logos, not B-value annotations.
- Input has no per-position conservation or SNP values yet — run the sequence analysis
  part (MUSCLE alignment, consensus construction with conservation threshold) first.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_2814
- http://edamontology.org/topic_0080
tools:
- name: Scop3D
  role: 'Implements the structure annotation: replaces PDB B-values with per-position
    percent variation/entropy (conservation analysis) or SNP frequency/type (SNP variants
    analysis)'
  repo: github.com/compomics/scop3d
- name: Biopython
  role: PDB parsing utilities used to load the structure and rewrite the B-value column
- name: BLAST
  role: 'Intermediate step: BLAST search of the consensus sequence against the PDB
    to retrieve a homologous structure when none is at hand'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:8614b369270c21f3f4d452c21696019c42a9e04f85f21e634d2e80cd460130f4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BLAST
  - Biopython
  - Scop3D
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_2814
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# conservation-metric-superimposition-on-3d-structure

## Summary

scop3D's structural annotation step rewrites the B-value column of a PDB file so each residue position carries its per-position sequence conservation metric — percent variation (100 minus percentage conservation) or information entropy for conservation analysis, or SNP frequency and SNP type for SNP variants analysis — enabling 3D visualization of sequence variation.

## When to use

Use when you have a multiple sequence alignment (or SNP matrix) with per-position conservation/entropy or SNP frequency/type values and a PDB structure on which to visualize them.

## When NOT to use

- No protein 3D structure is available or obtainable — the intermediate BLAST step against the PDB must first retrieve a homologous structure before annotation can run.
- You need visualization of absolute/relative amino acid occurrences rather than per-position mapping onto structure — those are delivered as text files and WebLogo sequence logos, not B-value annotations.
- Input has no per-position conservation or SNP values yet — run the sequence analysis part (MUSCLE alignment, consensus construction with conservation threshold) first.

## Inputs

- PDB file (from PDB database, user-supplied, or homologous structure found by BLASTing the consensus sequence against the PDB)
- Per-position conservation/variation values (percent variation, entropy factor) from a MUSCLE multiple sequence alignment of variant sequences
- Per-position SNP matrix (SNP frequency, SNP type) from Uniprot/Ensembl variant data or DNA sequence analysis
- Selected chain identifiers (optional, via -c flag)

## Outputs

- Two modified PDB files: first with B-values replaced by percent variation (or SNP frequency), second with B-values replaced by entropy factor (or SNP type)
- annotated_structure.pdb (validation target: parses as valid PDB, B-value fields carry substituted values)

## How to apply

Load the input PDB (retrieved from the PDB database, e.g. via BLAST of the consensus sequence, or user-supplied) with Biopython's PDB parsing utilities. Load the per-position value table produced by the upstream sequence analysis: percent variation (100 minus percentage conservation) or entropy factor after conservation analysis, or SNP frequency and SNP type after SNP variants analysis. Map each value onto the matching residue number in the PDB structure, optionally restricted to selected chains (e.g. chains A and C via the -c flag). Write two modified PDB files: the first replaces every atom's B-value with the primary metric (percent variation or SNP frequency), the second replaces it with the secondary metric (entropy factor or SNP type).

## Related tools

- **Scop3D** (Implements the structure annotation: replaces PDB B-values with per-position percent variation/entropy (conservation analysis) or SNP frequency/type (SNP variants analysis)) — github.com/compomics/scop3d
- **Biopython** (PDB parsing utilities used to load the structure and rewrite the B-value column)
- **BLAST** (Intermediate step: BLAST search of the consensus sequence against the PDB to retrieve a homologous structure when none is at hand)

## Examples

```
./scop3d structure -ent -w ~/outputdir -f 4F15.pdb -c A,C
```

## Evaluation signals

- annotated_structure.pdb exists in the working output directory (-w) and parses as a valid PDB file.
- B-value fields in the first output PDB carry the substituted percent variation (or SNP frequency) values, not the original B-factors.
- B-value fields in the second output PDB carry entropy factor (or SNP type) values.
- Per-position values align with residue numbers in the PDB; when chains are selected (e.g. -c A,C), only those chains are annotated.
- For conservation analysis, percent variation values correspond to 100 minus the percentage conservation at each position.

## Limitations

- The PDB structure must be obtainable or homologous: if no structure is at hand, scop3D relies on a BLAST search of the consensus sequence against the PDB, which may not yield a suitable match.
- The consensus residue used in the consensus sequence must be found above the user-set conservation threshold, which bounds how well-supported the per-position values are.
- Structure annotation depends on completing the sequence conservation or SNP variants analysis first; the output depends on which analysis was performed.

## Evidence

- [abstract] The output consists of two modified PDB files in which the B-values for each position are replaced by the percentage sequence conservation, or the information entropy for each position, respectively.: "The output consists of two modified PDB files in which the B-values for each position are replaced by the percentage sequence conservation, or the information entropy for each position, respectively."
