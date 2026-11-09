---
name: predicted-structure-output-validation
description: Use when a batch of ColabFold/AlphaFold2 structure predictions must be
  verified before curation or serving — e.g. confirming every predicted CBP60 protein
  structure has valid PDB/mmCIF and JSON-encoded pLDDT/PAE confidence files.
when_to_use_negative:
- When the input is raw amino acid sequence that has not yet been folded by ColabFold/AlphaFold2
  — use the structure prediction skill first, then validate outputs
- When the goal is interpreting or clustering structures by similarity (e.g. the TM-Align
  matrix-based clustering algorithm) rather than verifying file integrity and metric
  serialization
- When assessing biological accuracy of predictions (pLDDT/PAE confidence interpretation)
  rather than checking that prediction outputs exist, parse, and match the archive
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_0130
tools:
- name: ColabFold
  role: Generator of the AlphaFold2-predicted 3D structures (PDB/mmCIF) and confidence
    metrics (pLDDT, PAE) whose output files are validated
  repo: https://github.com/sokrypton/ColabFold
- name: AlphaFold2
  role: Deep-learning model underlying the predictions; produces the bfactor-embedded
    pLDDT values checked during validation
- name: CBP60-DB
  role: Reference archive of AlphaFold2-predicted CBP60 structures and JSON scoring
    metrics against which outputs are compared
  repo: https://github.com/KeaunAmani/cbp60db
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:c64c4027b82c836763b0428619db70f95dc5916953159670e4a3b407918b7c0e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/KeaunAmani/cbp60db
  repo_url: https://github.com/sokrypton/ColabFold
  tools:
  - AlphaFold2
  - CBP60-DB
  - ColabFold
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_0130
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# predicted-structure-output-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the outputs of ColabFold/AlphaFold2 structural prediction runs (e.g. CBP60-DB's plant CBP60 protein structures) by checking that each predicted structure file (PDB/mmCIF) and each pLDDT/PAE JSON file exists, parses correctly, and matches the archived AlphaFold2-predicted structures.

## When to use

Use when a batch of ColabFold/AlphaFold2 structure predictions must be verified before curation or serving — e.g. confirming every predicted CBP60 protein structure has valid PDB/mmCIF and JSON-encoded pLDDT/PAE confidence files.

## When NOT to use

- When the input is raw amino acid sequence that has not yet been folded by ColabFold/AlphaFold2 — use the structure prediction skill first, then validate outputs
- When the goal is interpreting or clustering structures by similarity (e.g. the TM-Align matrix-based clustering algorithm) rather than verifying file integrity and metric serialization
- When assessing biological accuracy of predictions (pLDDT/PAE confidence interpretation) rather than checking that prediction outputs exist, parse, and match the archive

## Inputs

- Predicted 3D structure files in PDB/mmCIF format (one per CBP60 protein from ColabFold/AlphaFold2)
- JSON files of per-residue pLDDT and pairwise PAE confidence metrics
- CBP60-DB archived AlphaFold2-predicted structures for the sampled proteins (reference set)
- CBP60 amino acid sequences selected for structure prediction

## Outputs

- Validation report confirming each PDB/mmCIF file exists and parses
- Validation report confirming each pLDDT/PAE JSON file exists and parses as valid JSON
- Concordance check of outputs against CBP60-DB archived AlphaFold2-predicted structures

## How to apply

After running ColabFold (the AlphaFold2 derivative) on a representative sample of CBP60 amino acid sequences, iterate over each predicted structure and perform two checks: (1) the structure file in PDB/mmCIF format exists and parses (e.g. atom/residue records readable, and the bfactor column carries pLDDT confidence values, higher = better); (2) the serialized JSON file containing per-residue pLDDT and pairwise PAE confidence metrics parses as valid JSON with the expected per-residue and residue-pair dimensions. Then compare the validated outputs against the CBP60-DB archived AlphaFold2-predicted structures for the sampled proteins to confirm concordance. The rationale is that CBP60-DB archives AlphaFold2 scoring metrics (pLDDT, PAE, pTM) as JSON alongside the MMseqs2 MSA in a3m format, so any file that fails to parse or mismatches the archive would break downstream structural clustering and the database's dedicated per-protein web pages.

## Related tools

- **ColabFold** (Generator of the AlphaFold2-predicted 3D structures (PDB/mmCIF) and confidence metrics (pLDDT, PAE) whose output files are validated) — https://github.com/sokrypton/ColabFold
- **AlphaFold2** (Deep-learning model underlying the predictions; produces the bfactor-embedded pLDDT values checked during validation)
- **CBP60-DB** (Reference archive of AlphaFold2-predicted CBP60 structures and JSON scoring metrics against which outputs are compared) — https://github.com/KeaunAmani/cbp60db

## Examples

```
git clone https://github.com/KeaunAmani/cbp60db.git && python -c "import ,glob; assert all(.load(open(f)) for f in glob.glob('*.'))"
```

## Evaluation signals

- Every predicted structure file (PDB/mmCIF) for the sampled CBP60 proteins exists and parses without read errors
- Every pLDDT/PAE JSON file exists and parses as valid JSON with per-residue (pLDDT) and residue-pair (PAE) entries
- Validated outputs match the CBP60-DB archived AlphaFold2-predicted structures for the same sampled proteins
- pLDDT values embedded in the bfactor column of PDB outputs are consistent with the JSON-serialized pLDDT metrics (higher = better)
- Accompanying MMseqs2 MSA files in a3m format are present and non-empty for each prediction run

## Limitations

- pLDDT confidence is stored in the bfactor column, which is incompatible with Molecular Replacement expectations (Phenix.phaser expects lower = better); misreading this breaks downstream crystallography use
- ColabFold GPU memory limits constrain predictions to roughly 2000 residues on ~16G GPUs, so very long CBP60 proteins may fail to produce structures to validate
- Validation only confirms file existence, parseability, and archive concordance — it does not establish that predictions are biologically correct

## Evidence

- [readme] pLDDT in bfactor column: [evidence span withheld — anchor status: not_found]
