---
name: plddt-pae-confidence-metric-extraction
description: Use when ColabFold (AlphaFold2) has produced predicted 3D structures
  for protein sequences (e.g., a plant protein family like CBP60) and you need to
  capture and serialize the model's self-estimated confidence (pLDDT, PAE, pTM) for
  downstream validation, display, or clustering.
when_to_use_negative:
- Input is raw, unaligned protein sequences with no predicted structure yet — run
  ColabFold first to generate the pLDDT/PAE data before extraction.
- You need experimentally determined (e.g., crystal or cryo-EM) structure quality
  metrics rather than AlphaFold2 self-estimated confidence — pLDDT/PAE are model-predicted,
  not experimental, measures.
- You only need structure similarity clustering rather than per-residue/pairwise confidence
  — use TM-Align-based clustering (as in CBP60-DB) instead.
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_2828
- http://edamontology.org/topic_0080
tools:
- name: ColabFold
  role: Generates AlphaFold2-predicted 3D structures and produces the pLDDT, PAE,
    and pTM confidence outputs that are extracted and serialized
  repo: https://github.com/sokrypton/ColabFold
- name: AlphaFold2
  role: Deep learning model underlying ColabFold whose confidence metrics (pLDDT,
    PAE, pTM) are archived per CBP60 protein
- name: CBP60-DB
  role: Repository/database that archives the extracted pLDDT/PAE JSON files alongside
    predicted structures for plant CBP60 proteins
  repo: https://github.com/KeaunAmani/cbp60db.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:4a3cae8048a4259a77ac12b424994653984cb1e30672228948b4a22b53a67341
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
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_2828
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# plddt-pae-confidence-metric-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-residue pLDDT and pairwise PAE (plus pTM) confidence metrics from ColabFold/AlphaFold2 structural predictions and archive them as JSON alongside the predicted PDB/mmCIF structures, as done for the plant CBP60 proteins in CBP60-DB.

## When to use

Use when ColabFold (AlphaFold2) has produced predicted 3D structures for protein sequences (e.g., a plant protein family like CBP60) and you need to capture and serialize the model's self-estimated confidence (pLDDT, PAE, pTM) for downstream validation, display, or clustering.

## When NOT to use

- Input is raw, unaligned protein sequences with no predicted structure yet — run ColabFold first to generate the pLDDT/PAE data before extraction.
- You need experimentally determined (e.g., crystal or cryo-EM) structure quality metrics rather than AlphaFold2 self-estimated confidence — pLDDT/PAE are model-predicted, not experimental, measures.
- You only need structure similarity clustering rather than per-residue/pairwise confidence — use TM-Align-based clustering (as in CBP60-DB) instead.

## Inputs

- CBP60 amino acid sequences (FASTA) from the CBP60-DB repository (github.com/KeaunAmani/cbp60db)
- ColabFold/AlphaFold2 prediction outputs (PDB or mmCIF structure files with pLDDT in the b-factor column)
- ColabFold result JSON containing pLDDT, PAE, and pTM data
- MMseqs2 MSA in a3m format

## Outputs

- Per-protein JSON files of AlphaFold2 confidence metrics (pLDDT, PAE, pTM score)
- Predicted 3D structures in PDB/mmCIF format
- MMseqs2 MSA files in a3m format
- JSON cluster map of predicted structures (as archived by CBP60-DB)

## How to apply

Run ColabFold (the AlphaFold2 derivative) on each selected amino acid sequence to generate highly accurate deep learning-predicted 3D structures, exporting each structure in PDB or mmCIF format. From each prediction result, extract the per-residue pLDDT values and the pairwise predicted aligned error (PAE), along with the pTM score, and serialize them as JSON files so they can be stored independently of the structural coordinate files. Retain the MMseqs2 multiple sequence alignment in a3m format, since the MSA underlies the prediction and its confidence interpretation. In CBP60-DB these metrics were archived for every predicted CBP60 protein structure so web pages could render confidence alongside structure. Note that in the PDB b-factor column ColabFold populates pLDDT (higher = better), which is useful for per-residue confidence visualization (e.g., coloring in PyMOL by pLDDT thresholds).

## Related tools

- **ColabFold** (Generates AlphaFold2-predicted 3D structures and produces the pLDDT, PAE, and pTM confidence outputs that are extracted and serialized) — https://github.com/sokrypton/ColabFold
- **AlphaFold2** (Deep learning model underlying ColabFold whose confidence metrics (pLDDT, PAE, pTM) are archived per CBP60 protein)
- **CBP60-DB** (Repository/database that archives the extracted pLDDT/PAE JSON files alongside predicted structures for plant CBP60 proteins) — https://github.com/KeaunAmani/cbp60db.git

## Examples

```
colabfold_batch --msa-mode mmseqs2_server cbp60_sequence.fasta out_dir/ && python -c "import ;d=.load(open('out_dir/ranked_0_scores.'));print(d['plddt'],d['ptm'])"
```

## Evaluation signals

- Each predicted structure file (PDB/mmCIF) and each corresponding pLDDT/PAE JSON file exists and parses correctly as valid JSON.
- The pLDDT values serialized in JSON match the b-factor column values in the corresponding PDB file for the same prediction.
- PAE matrix dimensions equal the number of residues in the predicted structure (N x N for N residues).
- Extracted outputs match the CBP60-DB archived AlphaFold2-predicted structures and metrics for the sampled CBP60 proteins.
- pLDDT values fall within the expected 0–100 range (used as color thresholds, e.g., coloring structure by pLDDT bands <50, <70, <90).

## Limitations

- Confidence metrics are AlphaFold2 self-estimates, not experimental quality measures; the b-factor column holds pLDDT (higher = better), which conflicts with crystallographic conventions where lower b-factor means better — relevant for molecular replacement (Phenix.phaser).
- Maximum sequence length depends on free Colab GPU memory (~2000 residues on ~16G GPUs), limiting which CBP60 transcripts can be predicted.
- MMseqs2 MSA server queries must be serial from a single IP if run locally, constraining batch prediction throughput.

## Evidence

- [methods] ColabFold confidence metrics archived for each predicted CBP60 protein structure: [evidence span withheld — anchor status: not_found]
- [methods] pLDDT/PAE serialized as JSON: [evidence span withheld — anchor status: not_found]
- [results] CBP60-DB archives AlphaFold2 scoring metrics in JSON with a3m MSA and cluster map: [evidence span withheld — anchor status: not_found]
- [readme] b-factor column populated with pLDDT: [evidence span withheld — anchor status: not_found]
- [methods] Validation of structure and metric files: [evidence span withheld — anchor status: not_found]
