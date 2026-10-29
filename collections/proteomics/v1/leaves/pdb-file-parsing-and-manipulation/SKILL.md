---
name: pdb-file-parsing-and-manipulation
description: Use when a per-position sequence table (conservation percentages, entropy
  factors, or SNP frequencies/types) must be projected onto a 3D protein structure
  stored as a PDB file, e.g. to color conservation onto a structure for visualization.
when_to_use_negative:
- The input is already a structure with per-residue annotations in its B-value column
  and no sequence-based metric needs to be projected
- The analysis goal is the sequence-conservation or SNP matrix itself (sequence analysis
  part), not its structural projection
- No PDB structure of the protein or a homolog can be identified and the BLAST search
  against the PDB yields no acceptable hit
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scop3D
  role: 'Performs the structure annotation: parses the PDB file and writes two modified
    PDB files with altered B-values (percent variation/entropy or SNP frequency/type)'
  repo: github.com/compomics/scop3d
- name: Biopython
  role: PDB parsing utilities used to load and manipulate the protein structure when
    substituting B-value fields
- name: BLAST
  role: Intermediate step to retrieve a homologous structure by searching the consensus
    sequence against the PDB when no structure is at hand
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:2bdc257ebb6081c56161cdc05441792fe7be0328aa884f19efef7f5164df0fa5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BLAST
  - Biopython
  - Scop3D
  - scop3D
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# pdb-file-parsing-and-manipulation

## Summary

Parse a protein PDB structure and rewrite its B-value column so that each residue position carries a per-position sequence metric (percentage conservation/variation, information entropy, SNP frequency, or SNP type), enabling 3D visualization of sequence conservation as in scop3D.

## When to use

Use when a per-position sequence table (conservation percentages, entropy factors, or SNP frequencies/types) must be projected onto a 3D protein structure stored as a PDB file, e.g. to color conservation onto a structure for visualization.

## When NOT to use

- The input is already a structure with per-residue annotations in its B-value column and no sequence-based metric needs to be projected
- The analysis goal is the sequence-conservation or SNP matrix itself (sequence analysis part), not its structural projection
- No PDB structure of the protein or a homolog can be identified and the BLAST search against the PDB yields no acceptable hit

## Inputs

- PDB file (retrieved from the PDB database, user-supplied, or identified via BLAST of the consensus sequence against the PDB)
- Per-position value table from upstream sequence analysis: percent variation / entropy factor (conservation analysis) or SNP frequency and SNP type (SNP variants analysis)
- Consensus FASTA sequence (consensus.fasta) for the intermediate BLAST step
- Chain selection string (e.g. A,C)

## Outputs

- First modified PDB file with B-values replaced by percent variation or SNP frequency per position
- Second modified PDB file with B-values replaced by entropy factor or SNP type per position
- Set of figures (snapshots of the protein from six distinct directions in space)

## How to apply

Obtain a PDB-file either from the PDB database or supplied by the user; if no structure exists, retrieve a homologous structure by BLASTing the consensus sequence (built by retaining the most frequently found residue per position across the MUSCLE multiple sequence alignment) against the PDB. Parse the structure with Biopython's PDB parsing utilities and map each per-position value from the upstream analysis table (percent variation = 100 minus percentage conservation, entropy factor, SNP frequency, or SNP type) onto the matching residue number, optionally restricting the mapping to selected chains via the -c flag (e.g. chains A,C). Write a first modified PDB file with the primary metric (percent variation for conservation analysis, SNP frequency for SNP variants analysis) replacing the B-value column of every atom, and a second modified PDB file with the secondary metric (entropy factor or SNP type) in the B-value column. Position coverage depends on the conservation threshold set by the user, since consensus residues must be found more frequently than this threshold.

## Related tools

- **scop3D** (Performs the structure annotation: parses the PDB file and writes two modified PDB files with altered B-values (percent variation/entropy or SNP frequency/type)) — github.com/compomics/scop3d
- **Biopython** (PDB parsing utilities used to load and manipulate the protein structure when substituting B-value fields)
- **BLAST** (Intermediate step to retrieve a homologous structure by searching the consensus sequence against the PDB when no structure is at hand)

## Examples

```
./scop3d structure -ent -w ~/outputdir -f 4F15.pdb -c A,C
```

## Evaluation signals

- annotated_structure.pdb (or equivalent output) exists in the output directory (-w) and parses as a valid PDB file
- The B-value column of the first output PDB carries percent variation (100 minus percentage conservation) or SNP frequency values rather than the original B-factors
- The B-value column of the second output PDB carries entropy factor or SNP type values per position
- Residue numbers in the modified PDB files match the positions in the upstream value table; chains are restricted to those given via -c (e.g. A,C) when specified
- Exactly two modified PDB-files are produced per structure annotation run, alongside the figure set

## Limitations

- The output depends on whether sequence conservation or SNP variants analysis was performed as the first step; the two PDB files carry different metrics in each mode
- Consensus residues must be found more often than the user-set conservation threshold, so positions below threshold may lack values to map
- Requires Python >= 2.7, Perl >= 5, Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss in the environment

## Evidence
