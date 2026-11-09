---
name: pdb-bvalue-structure-annotation
description: Use when per-position sequence metrics (SNP frequency, SNP type, percent
  variation, or entropy) must be visualized on a protein 3D structure by overwriting
  B-factors in a PDB file.
when_to_use_negative:
- No PDB structure exists and no homologous structure can be identified by BLASTing
  the consensus sequence against the PDB — structure annotation requires a PDB-file
- The goal is sequence-level analysis only (alignment, conservation/entropy matrices,
  WebLogo sequence logo) without mapping onto a 3D structure
- The input PDB already contains the desired annotation in its B-factor column and
  no recomputation of per-position values is needed
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_2814
- http://edamontology.org/topic_0092
tools:
- name: Scop3D
  role: 'Performs the structure annotation: loads the PDB, maps per-position values
    onto residue numbers, and writes the two modified PDB files with altered B-values'
  repo: github.com/compomics/scop3d
- name: Biopython
  role: PDB parsing utilities used to load the input structure and write the B-value-substituted
    PDB files
- name: BLAST
  role: 'Intermediate step: BLASTs the consensus sequence against the PDB to retrieve
    a homologous structure when no PDB file is at hand'
- name: MUSCLE
  role: Aligns variant sequences upstream so the per-position SNP frequency/type matrix
    feeding the structure annotation can be built
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:94905a93cce39f93894399a0588cc0ee3469250067f63e025024d28947aa0f23
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BLAST
  - Bioperl
  - Biopython
  - MUSCLE
  - Scop3D
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_2814
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# pdb-bvalue-structure-annotation

## Summary

Annotate a protein 3D structure by replacing the B-value (B-factor) column of a PDB file with per-position sequence metrics — SNP frequency and SNP type for SNP variants analysis, or percent variation and entropy for conservation analysis — producing two modified PDB files.

## When to use

Use when per-position sequence metrics (SNP frequency, SNP type, percent variation, or entropy) must be visualized on a protein 3D structure by overwriting B-factors in a PDB file.

## When NOT to use

- No PDB structure exists and no homologous structure can be identified by BLASTing the consensus sequence against the PDB — structure annotation requires a PDB-file
- The goal is sequence-level analysis only (alignment, conservation/entropy matrices, WebLogo sequence logo) without mapping onto a 3D structure
- The input PDB already contains the desired annotation in its B-factor column and no recomputation of per-position values is needed

## Inputs

- PDB structure file (from the PDB database, user-supplied, or retrieved by BLAST of consensus.fasta against the PDB)
- Per-position value table from upstream sequence analysis (SNP frequency and SNP type, or percent variation and entropy factor)
- consensus.fasta from the sequence analysis step (used as BLAST input if no structure is at hand)

## Outputs

- First modified PDB file with B-values replaced by the primary metric (SNP frequency or percent variation)
- Second modified PDB file with B-values replaced by the secondary metric (SNP type or entropy factor)
- Set of figures (snapshots of the protein from six distinct directions in space)

## How to apply

First run the upstream sequence analysis (conservation or SNP variants) so scop3D produces the per-position value tables and a consensus.fasta; the SNP variants analysis for a UniProt protein such as P06858 downloads variant data from UniProt and Ensembl, aligns sequences with MUSCLE, and builds a matrix annotating the protein sequence with the frequency, variants, and type of each variant. Obtain a PDB file either supplied by the user, downloaded from the PDB, or retrieved via a BLAST search of the consensus sequence against the PDB. Load the PDB with Biopython parsing utilities and map each per-position value onto the matching residue number, optionally restricted to selected chains (e.g. chains A and C via the -c flag). Write two modified PDB files: for SNP variants analysis, the first PDB carries SNP frequency in the B-value column and the second carries SNP type; for conservation analysis, the first carries percent variation (100 minus percentage conservation) and the second the entropy factor. Validate that two altered PDB files appear in the output directory with B-value fields carrying the substituted per-position values rather than the original B-factors.

## Related tools

- **Scop3D** (Performs the structure annotation: loads the PDB, maps per-position values onto residue numbers, and writes the two modified PDB files with altered B-values) — github.com/compomics/scop3d
- **Biopython** (PDB parsing utilities used to load the input structure and write the B-value-substituted PDB files)
- **BLAST** (Intermediate step: BLASTs the consensus sequence against the PDB to retrieve a homologous structure when no PDB file is at hand)
- **MUSCLE** (Aligns variant sequences upstream so the per-position SNP frequency/type matrix feeding the structure annotation can be built)

## Examples

```
./scop3d structure -var -w ~/outputdir -i 4ACQ
```

## Evaluation signals

- Two modified PDB files with altered B-values are present in the output directory (-w ~/outputdir)
- annotated_structure.pdb parses as a valid PDB file and its B-value fields carry the substituted per-position values rather than the original B-factors
- For SNP variants runs, first PDB B-values equal SNP frequency per position and second PDB B-values equal SNP type per position
- For conservation runs, first PDB B-values equal percent variation (100 minus percentage conservation) and second PDB B-values equal the entropy factor
- With -c A,C only chains A and C of the input structure carry the substituted B-values

## Limitations

- The output depends on whether sequence conservation or SNP variants analysis was performed as the first step — mixing up the two metrics in the output PDBs indicates the wrong upstream mode was run
- A PDB-file is required; if none is available, one must be obtained from the PDB or via BLAST of the consensus sequence against the PDB
- Consensus residues must be found more often than the user-set conservation threshold, affecting which positions receive values upstream

## Evidence
