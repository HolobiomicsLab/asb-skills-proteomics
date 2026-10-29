---
name: sequence-position-to-structure-residue-mapping
description: Use when per-position sequence conservation, entropy, or SNP frequency/type
  values from a multiple sequence alignment must be visualized on a protein 3D structure
  by rewriting PDB B-value columns.
when_to_use_negative:
- No protein structure is available and no homologous structure can be retrieved by
  BLASTing the consensus sequence against the PDB — structural annotation requires
  a PDB-file
- Only qualitative residue-level inspection of a sequence logo is needed; the alignment/consensus/abundance
  matrices and WebLogo logo step suffice without structure annotation
- The per-position metrics are not residue-resolved (e.g. whole-sequence statistics),
  so they cannot be mapped to matching residue numbers in the PDB
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: scop3D
  role: 'Performs the structure annotation step: substitutes B-values in two output
    PDB-files with per-position conservation/entropy or SNP frequency/type values'
  repo: github.com/compomics/scop3d
- name: Biopython
  role: PDB parsing utilities used to load and rewrite the structure's B-value column
    per residue
- name: BLAST
  role: Intermediate step to retrieve a homologous PDB structure when none is at hand,
    by searching the consensus sequence against the PDB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201400354
    title: 'Scop3D: Three‐dimensional visualization of sequence conservation'
  source_content_hash: sha256:c7fa147ad715ac95bb3173754f575d68ead5382a17777b5f37032f91720e9dd3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BLAST
  - Biopython
  - Scop3D
  - scop3D
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# sequence-position-to-structure-residue-mapping

## Summary

Maps per-position sequence conservation, information entropy, or SNP frequency/type values computed from a multiple sequence alignment back onto a protein 3D structure by substituting each residue's B-value in PDB output files, enabling 3D visualization of sequence variation as implemented in scop3D.

## When to use

Use when per-position sequence conservation, entropy, or SNP frequency/type values from a multiple sequence alignment must be visualized on a protein 3D structure by rewriting PDB B-value columns.

## When NOT to use

- No protein structure is available and no homologous structure can be retrieved by BLASTing the consensus sequence against the PDB — structural annotation requires a PDB-file
- Only qualitative residue-level inspection of a sequence logo is needed; the alignment/consensus/abundance matrices and WebLogo logo step suffice without structure annotation
- The per-position metrics are not residue-resolved (e.g. whole-sequence statistics), so they cannot be mapped to matching residue numbers in the PDB

## Inputs

- PDB structure file (from the PDB database, user-supplied, or via BLAST of the consensus sequence against the PDB)
- Per-position metric table from sequence conservation analysis (percent variation / entropy factor)
- Per-position SNP table from SNP variants analysis (SNP frequency, variants, type)
- Consensus sequence (consensus.fasta) for BLAST-based structure retrieval

## Outputs

- Two modified PDB files with B-values replaced by primary and secondary per-position metrics (percent variation and entropy factor, or SNP frequency and SNP type)
- Annotated structure figures (a set of figures accompanying the two PDB-files)

## How to apply

Load the target PDB structure (retrieved from the PDB database, user-supplied, or found by BLASTing the consensus sequence against the PDB) using Biopython's PDB parsing utilities. Load the per-position value table from the upstream sequence analysis: percent variation (100 minus percentage conservation) and entropy factor for conservation analysis, or SNP frequency and SNP type for SNP variants analysis. Map each value onto the matching residue number in the structure, optionally restricted to selected chains (e.g. chains A and C via the -c flag). Write two modified PDB files: for conservation analysis the first PDB-file has B-values replaced by percent variation and the second by the entropy factor; for SNP variants the first uses SNP frequency and the second SNP type. Validate that the annotated structure file exists, parses as a valid PDB, and that B-value fields carry the substituted per-position values rather than the original B-factors.

## Related tools

- **scop3D** (Performs the structure annotation step: substitutes B-values in two output PDB-files with per-position conservation/entropy or SNP frequency/type values) — github.com/compomics/scop3d
- **Biopython** (PDB parsing utilities used to load and rewrite the structure's B-value column per residue)
- **BLAST** (Intermediate step to retrieve a homologous PDB structure when none is at hand, by searching the consensus sequence against the PDB)

## Examples

```
./scop3d structure -ent -w ~/outputdir -f 4F15.pdb -c A,C
```

## Evaluation signals

- annotated_structure.pdb exists in the output directory and parses as a valid PDB file
- B-value fields in the first and second modified PDB files carry the substituted per-position values (percent variation / SNP frequency and entropy factor / SNP type), not the original B-factors
- B-values in the output match the upstream per-position matrix (relative abundance percent or SNP frequency/type matrix) at the corresponding residue numbers
- When the -c flag is used (e.g. -c A,C), only the specified chains' residues are annotated
- Values fall in expected ranges: percent variation between 0 and 100 (one hundred minus percentage conservation); entropy factor consistent with the sequence alignment

## Limitations

- Structural annotation requires a PDB-file; if none exists, a homologous structure must be retrieved via BLAST of the consensus sequence against the PDB, which depends on the availability and quality of homologous structures
- The consensus residue at each position must be found more often than the user-set conservation threshold, so low-confidence consensus positions affect the mapped values
- Output semantics depend on which upstream analysis was run (conservation vs SNP variants), so the two output PDB files carry different metrics in each case
- Tool requires Python >= 2.7, Perl >= 5, Biopython, Bioperl, Ensembl Perl API, Weblogo, Muscle, and Emboss

## Evidence
