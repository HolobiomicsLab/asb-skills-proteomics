---
name: phylogenetic-tree-inference
description: Use when you have amino acid sequences of a protein family (e.g., FASTA
  of homologs across many genomes) and need an evolutionary tree relating them, starting
  from an unaligned FASTA.
when_to_use_negative:
- Your input is already an alignment or a finished Newick tree — re-running MUSCLE
  alignment is unnecessary and FastTree would need the aligned FASTA, not a tree
- You need exhaustive maximum-likelihood inference with bootstrap support or model
  selection (e.g., RAxML/IQ-TREE use cases) — FastTree provides an approximate ML
  topology only
- The goal is structural comparison rather than sequence-based phylogeny — CBP60-DB
  uses TM-Align-based structural clustering for that purpose instead
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0080
tools:
- name: MUSCLE
  role: Generates the multiple sequence alignment (muscle_alignment.fasta) that serves
    as FastTree's input
- name: MEGA11
  role: Provides the configuration parameterization for the MUSCLE alignment (Table
    S1)
- name: FastTree
  role: Infers the approximate maximum-likelihood phylogeny from the MUSCLE alignment
    and outputs it in Newick format
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:e9588a45c00def36d027f52eeddc2ec31dd75a7365887ca3437230f4ab6239a1
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# phylogenetic-tree-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a phylogeny of a protein family by feeding a MUSCLE-generated multiple sequence alignment (produced with MEGA11 configuration) into FastTree, yielding an approximate maximum-likelihood tree in Newick format. In CBP60-DB this was applied to 1996 unique CBP60 proteins from 62 plant genomes.

## When to use

Use when you have amino acid sequences of a protein family (e.g., FASTA of homologs across many genomes) and need an evolutionary tree relating them, starting from an unaligned FASTA.

## When NOT to use

- Your input is already an alignment or a finished Newick tree — re-running MUSCLE alignment is unnecessary and FastTree would need the aligned FASTA, not a tree
- You need exhaustive maximum-likelihood inference with bootstrap support or model selection (e.g., RAxML/IQ-TREE use cases) — FastTree provides an approximate ML topology only
- The goal is structural comparison rather than sequence-based phylogeny — CBP60-DB uses TM-Align-based structural clustering for that purpose instead

## Inputs

- Amino acid FASTA file of CBP60 family proteins (1996 unique proteins from 62 plant genomes) retrieved from CBP60-DB
- MUSCLE alignment configuration parameters defined in MEGA11 (Table S1)

## Outputs

- muscle_alignment.fasta — MUSCLE multiple sequence alignment of the CBP60 proteins
- cbp60_tree.nwk — approximate maximum-likelihood phylogeny in Newick format with CBP60 protein tip labels

## How to apply

Retrieve the family's amino acid sequences (here 1996 unique CBP60 proteins across 62 phylogenetically diverse plant genomes) from the resource database. Generate a multiple sequence alignment with MUSCLE, parameterizing the alignment configuration in MEGA11 as documented in Table S1, and write the result to muscle_alignment.fasta. Run FastTree on that alignment to infer an approximate maximum-likelihood phylogeny, which is scalable to thousands of sequences. Save the inferred tree in Newick format as cbp60_tree.nwk and validate that it parses and its tip labels match the proteins in the alignment.

## Related tools

- **MUSCLE** (Generates the multiple sequence alignment (muscle_alignment.fasta) that serves as FastTree's input)
- **MEGA11** (Provides the configuration parameterization for the MUSCLE alignment (Table S1))
- **FastTree** (Infers the approximate maximum-likelihood phylogeny from the MUSCLE alignment and outputs it in Newick format)

## Examples

```
FastTree muscle_alignment.fasta > cbp60_tree.nwk
```

## Evaluation signals

- cbp60_tree.nwk exists and is non-empty after the FastTree run
- cbp60_tree.nwk parses as valid Newick
- Tip labels in cbp60_tree.nwk correspond to the CBP60 proteins present in muscle_alignment.fasta
- The number of taxa in the Newick tree matches the number of sequences in the alignment (1996 unique CBP60 proteins)
- muscle_alignment.fasta contains all input sequences with aligned (gap-inserted) rows of equal length

## Limitations

- FastTree yields an approximate maximum-likelihood tree; the workflow does not describe bootstrap support or substitution-model selection
- Alignment quality depends on the MUSCLE/MEGA11 configuration (Table S1); no alternative alignment settings were evaluated in this workflow
- The workflow covers sequence-based phylogeny; kingdom-wide structural similarity was assessed separately with a TM-Align-based clustering algorithm, not by this tree

## Evidence

- [full_text] The MUSCLE-generated alignment FASTA file was used as input to the FastTree algorithm, which produced a phylogenetic tree of CBP60 proteins in the Newick file format.: [evidence span withheld — anchor status: not_found]
- [supplementary] Table S1: Configuration used for MUSCLE alignment generation using MEGA11.: "Table S1: Configuration used for MUSCLE alignment generation using MEGA11."
