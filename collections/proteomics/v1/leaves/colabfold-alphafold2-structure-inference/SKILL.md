---
name: colabfold-alphafold2-structure-inference
description: Use when you have amino acid sequences for a protein family and need
  predicted 3D structures plus confidence metrics (pLDDT, PAE, pTM) without running
  full AlphaFold2 locally.
when_to_use_negative:
- You need experimentally determined structures rather than deep learning predictions
- Your sequences exceed ColabFold limits — with ~16G free Colab GPUs the max length
  is ~2000 residues
- You require native AlphaFold2 jackhmmer MSAs against full databases rather than
  the MMseqs2 MSA server (MMseqs2 queries from multiple computers on one IP are discouraged)
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_2275
tools:
- name: ColabFold
  role: AlphaFold2 derivative used to generate highly accurate deep learning-predicted
    3D structures from CBP60 amino acid sequences, using the MMseqs2 MSA server
  repo: https://github.com/sokrypton/ColabFold
- name: AlphaFold2
  role: Underlying deep learning model whose predictions, including pLDDT/PAE/pTM
    scoring metrics, are archived in CBP60-DB for all plant CBP60 proteins
- name: CBP60-DB
  role: Repository providing the CBP60 amino acid sequences (>2,000 proteins) that
    serve as prediction inputs and archiving the resulting structures and confidence
    metrics
  repo: https://github.com/KeaunAmani/cbp60db
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:158941e7e04bb95f95f96b95e3754a485adfdab8627d5c327271e1c9452f150a
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
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_2275
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# colabfold-alphafold2-structure-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run ColabFold (an AlphaFold2 derivative) on protein amino acid sequences to generate deep learning-predicted 3D structures with per-residue pLDDT and pairwise PAE confidence metrics, as done to predict structures for the >2,000 plant CBP60 proteins archived in CBP60-DB.

## When to use

Use when you have amino acid sequences for a protein family and need predicted 3D structures plus confidence metrics (pLDDT, PAE, pTM) without running full AlphaFold2 locally.

## When NOT to use

- You need experimentally determined structures rather than deep learning predictions
- Your sequences exceed ColabFold limits — with ~16G free Colab GPUs the max length is ~2000 residues
- You require native AlphaFold2 jackhmmer MSAs against full databases rather than the MMseqs2 MSA server (MMseqs2 queries from multiple computers on one IP are discouraged)

## Inputs

- Amino acid sequences of target proteins (e.g., CBP60 family sequences in FASTA form from CBP60-DB)
- Optional: ColabFold notebook selection (AlphaFold2_mmseqs2 for monomers/complexes, AlphaFold2_batch for batch prediction)

## Outputs

- Predicted 3D structures in PDB/mmCIF format (B-factor column populated with pLDDT values)
- Per-residue pLDDT and pairwise PAE confidence metrics serialized as JSON files
- MMseqs2 multiple sequence alignments in a3m format
- pTM scores (when computed for the prediction)

## How to apply

Retrieve the amino acid sequences of the target proteins (e.g., the more than 2,000 CBP60 family proteins in CBP60-DB from github.com/KeaunAmani/cbp60db), select representative sequences for prediction, and run ColabFold — the AlphaFold2 derivative that generates highly accurate predictions via the MMseqs2 MSA server — on each sequence. Export each predicted structure in PDB/mmCIF format, where the B-factor column is populated with per-residue pLDDT confidence values (higher = better). Extract the per-residue pLDDT and pairwise PAE confidence metrics from each prediction and serialize them as JSON files alongside the MMseqs2 MSA in a3m format. Confidence thresholds are meaningful for interpretation: pLDDT < 50 indicates low confidence (pymol can render structures colored by pLDDT with the red_yellow_green_cyan_blue spectrum from 50 to 90).

## Related tools

- **ColabFold** (AlphaFold2 derivative used to generate highly accurate deep learning-predicted 3D structures from CBP60 amino acid sequences, using the MMseqs2 MSA server) — https://github.com/sokrypton/ColabFold
- **AlphaFold2** (Underlying deep learning model whose predictions, including pLDDT/PAE/pTM scoring metrics, are archived in CBP60-DB for all plant CBP60 proteins)
- **CBP60-DB** (Repository providing the CBP60 amino acid sequences (>2,000 proteins) that serve as prediction inputs and archiving the resulting structures and confidence metrics) — https://github.com/KeaunAmani/cbp60db

## Examples

```
git clone https://github.com/KeaunAmani/cbp60db.git  # retrieve CBP60 sequences, then run ColabFold AlphaFold2_mmseqs2 notebook per sequence
```

## Evaluation signals

- Each predicted structure file (PDB/mmCIF) and each pLDDT/PAE JSON file exists and parses correctly
- Predicted structures and metrics match the CBP60-DB archived AlphaFold2-predicted structures for the sampled proteins
- The B-factor column of each output PDB is populated with pLDDT confidence values (higher = better), not conventional b-factors
- MMseqs2 MSA output is present in a3m format for each predicted sequence

## Limitations

- Maximum sequence length depends on the free GPU provided by Google Colab; for GPUs with ~16G the max length is ~2000 residues
- B-factor column contains pLDDT values, which is incompatible with tools like Phenix.phaser expecting a 'real' b-factor (lower = better)
- The MMseqs2 MSA server should only be queried serially from a single IP when used from a local computer

## Evidence

- [intro] ColabFold is an AlphaFold2 derivative that addresses the aforementioned issues with AlphaFold2 and is able to generate highly accurate predictions: "ColabFold (Mirdita et al., 2022) is an AlphaFold2 derivative that addresses the aforementioned issues with AlphaFold2 and is able to generate highly accurate predictions"
- [other] For each predicted CBP60 protein structure, CBP60-DB archives AlphaFold2 scoring metrics (pLDDT, PAE, and pTM score) in JSON format, along with the MMseqs2 MSA in a3m format: [evidence span withheld — anchor status: not_found]
- [readme] the bfactor column is populated with pLDDT confidence values (higher = better). Phenix.phaser expects a "real" bfactor, where (lower = better): [evidence span withheld — anchor status: not_found]
- [readme] Limits depends on free GPU provided by Google Colab `fingers-crossed` - For GPUs with ~16G the max length is ~2000: [evidence span withheld — anchor status: not_found]
