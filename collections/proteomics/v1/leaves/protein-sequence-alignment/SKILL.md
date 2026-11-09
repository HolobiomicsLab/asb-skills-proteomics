---
name: protein-sequence-alignment
description: Use when a set of homologous protein amino acid sequences (e.g. a protein
  family retrieved from a database like CBP60-DB) must be aligned before phylogenetic
  or conservation analysis.
when_to_use_negative:
- The goal is 3D structural comparison rather than sequence alignment — CBP60-DB uses
  TM-Align on AlphaFold2-predicted structures for structural clustering instead
- Input is nucleotide/cDNA sequence requiring codon-aware alignment rather than protein-level
  MUSCLE alignment
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: MUSCLE
  role: Generates the multiple sequence alignment of CBP60 proteins from retrieved
    amino acid sequences
- name: MEGA11
  role: Provides the configuration used for MUSCLE alignment generation (Table S1)
- name: FastTree
  role: Consumes the MUSCLE alignment FASTA to infer the phylogenetic tree (Newick
    format)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:028c11ad67b14f903bea5f35dbe9d07d0b2e29fba002a7cc7a7a5f8a0dc8ad09
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/KeaunAmani/cbp60db
  tools:
  - FastTree
  - MEGA11
  - MUSCLE
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# protein-sequence-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a multiple sequence alignment of the CBP60 protein family (1996 unique proteins across 62 plant genomes) using MUSCLE with parameters configured in MEGA11, producing a FASTA alignment that downstream phylogenetic inference (FastTree) consumes.

## When to use

Use when a set of homologous protein amino acid sequences (e.g. a protein family retrieved from a database like CBP60-DB) must be aligned before phylogenetic or conservation analysis.

## When NOT to use

- The goal is 3D structural comparison rather than sequence alignment — CBP60-DB uses TM-Align on AlphaFold2-predicted structures for structural clustering instead
- Input is nucleotide/cDNA sequence requiring codon-aware alignment rather than protein-level MUSCLE alignment

## Inputs

- CBP60 amino acid sequences (1996 unique proteins from 62 plant genomes, FASTA format) retrieved from CBP60-DB
- MUSCLE alignment configuration parameterized in MEGA11 (Table S1)

## Outputs

- muscle_alignment.fasta — multiple sequence alignment of CBP60 proteins in FASTA format

## How to apply

Retrieve the CBP60 amino acid sequences (1996 unique proteins across 62 phylogenetically diverse plant genomes) from CBP60-DB. Run MUSCLE with the alignment configuration parameterized in MEGA11 (documented in Table S1 of the supplement) to produce a multiple sequence alignment of the protein family. Write the alignment to muscle_alignment.fasta so it can be passed directly to FastTree for approximate maximum-likelihood phylogeny inference. Validate the alignment before downstream use by confirming it parses as valid FASTA and that its sequence identifiers match the CBP60 proteins retrieved from the database.

## Related tools

- **MUSCLE** (Generates the multiple sequence alignment of CBP60 proteins from retrieved amino acid sequences)
- **MEGA11** (Provides the configuration used for MUSCLE alignment generation (Table S1))
- **FastTree** (Consumes the MUSCLE alignment FASTA to infer the phylogenetic tree (Newick format))

## Evaluation signals

- muscle_alignment.fasta exists and parses as valid FASTA
- Alignment contains all 1996 unique CBP60 protein sequences retrieved from the 62 plant genomes
- All aligned sequences are the same length (padded with gap characters) and identifiers match the input protein set
- Downstream FastTree run on muscle_alignment.fasta produces a valid Newick tree (cbp60_tree.nwk) whose tip labels correspond to proteins in the alignment

## Limitations

- Alignment configuration is tied to the MEGA11 parameterization in Table S1; other MUSCLE settings are not documented in this paper
- MUSCLE produces a sequence-level alignment only; functional inference of conserved structure in CBP60-DB is done separately via AlphaFold2/ColabFold structures and TM-Align clustering

## Evidence

- [other] FastTree consumes alignment: [evidence span withheld — anchor status: not_found]
