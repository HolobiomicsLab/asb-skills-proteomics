---
name: multiple-sequence-alignment-generation
description: Use when amino acid sequences of a protein family (e.g., FASTA of CBP60
  homologs) need a multiple sequence alignment as input for phylogenetic tree construction
  such as FastTree.
when_to_use_negative:
- Input is nucleotide/cDNA rather than the protein amino acid sequences used for the
  CBP60 family alignment
- The goal is 3D structural similarity clustering rather than sequence-based phylogeny
  — the article's clustering algorithm uses TM-Align structural matrices, not MUSCLE
  alignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MUSCLE
  role: Generates the multiple sequence alignment of CBP60 amino acid sequences; configuration
    parameterized via MEGA11
- name: MEGA11
  role: Hosts the alignment configuration (Table S1) under which the MUSCLE alignment
    was generated
- name: FastTree
  role: Consumes the MUSCLE alignment (muscle_alignment.fasta) to infer the approximate
    maximum-likelihood phylogeny
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:064c786e74490b3a4fb7ac4690e39d3a077b641b04edca4dd717a4dc5be17d38
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multiple-sequence-alignment-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a multiple sequence alignment (MSA) of a protein family's amino acid sequences using MUSCLE, with alignment configuration parameterized in MEGA11, to feed downstream phylogenetic inference. In CBP60-DB this produced muscle_alignment.fasta for the 1996 unique CBP60 proteins from 62 plant genomes.

## When to use

Use when amino acid sequences of a protein family (e.g., FASTA of CBP60 homologs) need a multiple sequence alignment as input for phylogenetic tree construction such as FastTree.

## When NOT to use

- Input is nucleotide/cDNA rather than the protein amino acid sequences used for the CBP60 family alignment
- The goal is 3D structural similarity clustering rather than sequence-based phylogeny — the article's clustering algorithm uses TM-Align structural matrices, not MUSCLE alignments

## Inputs

- FASTA file of amino acid sequences for the protein family (e.g., 1996 unique CBP60 proteins)
- MEGA11 alignment configuration (Table S1 parameterization for MUSCLE)

## Outputs

- muscle_alignment.fasta — MUSCLE multiple sequence alignment in FASTA format
- downstream cbp60_tree.nwk — Newick phylogeny inferred by FastTree from the alignment

## How to apply

Retrieve the curated amino acid sequences (in the CBP60-DB case, 1996 unique proteins across 62 phylogenetically diverse plant genomes). Run MUSCLE to produce the multiple sequence alignment, adopting the alignment configuration parameterized in MEGA11 as documented in Table S1 of the article's supplementary material. Write the aligned sequences to muscle_alignment.fasta in FASTA format. The alignment can then serve as direct input to FastTree for approximate maximum-likelihood phylogeny inference. Validate by confirming the output parses as valid FASTA and retains one record per input protein, with tip labels matching the source proteins.

## Related tools

- **MUSCLE** (Generates the multiple sequence alignment of CBP60 amino acid sequences; configuration parameterized via MEGA11)
- **MEGA11** (Hosts the alignment configuration (Table S1) under which the MUSCLE alignment was generated)
- **FastTree** (Consumes the MUSCLE alignment (muscle_alignment.fasta) to infer the approximate maximum-likelihood phylogeny)

## Examples

```
muscle -in cbp60_proteins.fasta -out muscle_alignment.fasta
```

## Evaluation signals

- muscle_alignment.fasta exists and parses as valid FASTA
- The number of aligned records equals the number of input proteins (1996 unique CBP60 proteins in the reference case)
- Downstream cbp60_tree.nwk exists, parses as valid Newick, and contains tip labels corresponding to the CBP60 proteins in the alignment
- Alignment parameters match the MEGA11 configuration recorded in Table S1 of the supplementary material

## Limitations

- The alignment is sequence-based; it does not capture structural similarity, which the article addresses separately with a TM-Align matrix-based clustering algorithm
- Alignment configuration is only documented indirectly via Table S1 in the supplementary material, so exact parameter reproduction depends on that table
- The pipeline was demonstrated on the CBP60 protein family; scaling behavior to other protein family sizes is not characterized in the article

## Evidence

- [supplementary] MEGA11 configuration for MUSCLE alignment: "Table S1: Configuration used for MUSCLE alignment generation using MEGA11."
- [abstract] Scale of the protein set aligned: "1052 CBP60 gene homologs (encoding 2376 unique transcripts and 1996 unique proteins) across 62 phylogenetically diverse genomes"
- [readme] Size and scope of the sequence database feeding the alignment: "This database features more than 2,000 proteins of the CBP60 family in the plant kingdom. Deep learning-predicted structures, cDNA/amino acid sequences, species of origin"
