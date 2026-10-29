---
name: cbp60-family-sequence-curation
description: Use when building a reference database of all homologs of a gene family
  (e.g. CBP60) across phylogenetically diverse published plant genomes and you need
  verified counts of unique genes, transcripts, and proteins.
when_to_use_negative:
- The target species lacks a published genome in Gramene — homolog recovery will be
  incomplete relative to the kingdom-wide scope
- You need experimentally validated structures or curated annotations rather than
  comprehensively collected homolog sequences
- The family of interest is already fully represented in an existing curated database,
  making re-collection and deduplication redundant
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: Gramene
  role: Comparative genomics resource queried to identify CBP60 gene homologs and
    download corresponding gene, cDNA, and protein sequences across plant species
    with published genomes
- name: ColabFold
  role: 'Downstream consumer: AlphaFold2 derivative used to generate predicted 3D
    structures from the curated CBP60 protein sequences archived in CBP60-DB'
  repo: sokrypton/ColabFold
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:a91d8a7790c79b7a3e45e39bd17429ce611b0fdd369dc2e2c5b5511fb6843637
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
  - ColabFold
  - Gramene
  - MEGA11
  - MUSCLE
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cbp60-family-sequence-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates a kingdom-wide protein family sequence dataset by querying the Gramene comparative genomics website for homologs across plant species with published genomes, downloading gene/cDNA/protein sequences, and deduplicating to yield category-specific counts (1052 genes, 2376 transcripts, 1996 proteins across 62 genomes for CBP60).

## When to use

Use when building a reference database of all homologs of a gene family (e.g. CBP60) across phylogenetically diverse published plant genomes and you need verified counts of unique genes, transcripts, and proteins.

## When NOT to use

- The target species lacks a published genome in Gramene — homolog recovery will be incomplete relative to the kingdom-wide scope
- You need experimentally validated structures or curated annotations rather than comprehensively collected homolog sequences
- The family of interest is already fully represented in an existing curated database, making re-collection and deduplication redundant

## Inputs

- Gramene comparative genomics query results for CBP60 gene homologs (https://www.gramene.org)
- Gene, cDNA/transcript, and protein FASTA sequences downloaded per homolog
- List of plant species with published genomes

## Outputs

- Deduplicated cDNA/transcript sequence set (2376 unique transcripts for CBP60)
- Deduplicated protein sequence set (1996 unique proteins for CBP60)
- Verified count table (e.g. cbp60_sequence_counts.csv: 1052 genes, 2376 transcripts, 1996 proteins, 62 genomes)

## How to apply

Query the Gramene comparative genomics website (https://www.gramene.org) to identify homologs of the target family across plant species with published genomes, rather than relying on BLAST against a single reference. For each homolog, download the gene, cDNA/transcript, and protein sequences from Gramene so that all three sequence classes are captured per locus. Deduplicate transcripts and protein sequences separately, since one gene can encode multiple transcripts but multiple transcripts may collapse to one unique protein (1052 genes → 2376 transcripts → 1996 proteins in the CBP60 case). Tally totals per category along with the number of source genomes (62 for CBP60-DB) and compare each against reported values before writing the counts, e.g. to cbp60_sequence_counts.csv. These curated sequences then feed downstream steps such as MUSCLE alignment/MEGA11 configuration and ColabFold (AlphaFold2) structural prediction archived in CBP60-DB.

## Related tools

- **Gramene** (Comparative genomics resource queried to identify CBP60 gene homologs and download corresponding gene, cDNA, and protein sequences across plant species with published genomes)
- **ColabFold** (Downstream consumer: AlphaFold2 derivative used to generate predicted 3D structures from the curated CBP60 protein sequences archived in CBP60-DB) — sokrypton/ColabFold

## Examples

```
python tally_cbp60_counts.py --genes genes.fa --cdna cdna.fa --proteins proteins.fa --out cbp60_sequence_counts.csv  # expect 1052/2376/1996/62
```

## Evaluation signals

- Gene homolog count matches 1052
- Unique transcript count matches 2376 after deduplication
- Unique protein count matches 1996 after deduplication
- Number of distinct source plant genomes matches 62
- Count table (e.g. cbp60_sequence_counts.csv) exists and each category total is consistent with the upstream sequence FASTA files

## Limitations

- Coverage is restricted to plant species with published genomes available through Gramene; homologs in unsequenced taxa are missed
- Deduplication at the transcript and protein levels means counts differ per category, so reporting a single 'number of sequences' would be misleading
- The collection reflects a snapshot of Gramene content and may need updating as new plant genomes are published

## Evidence

- [abstract] 1052 gene homologs collected via Gramene: "comprehensively characterized 1052 CBP60 gene homologs (encoding 2376 unique transcripts and 1996 unique proteins) across 62 phylogenetically diverse genomes in the plant kingdom"
- [readme] Gramene as the collection source: "This database features more than 2,000 proteins of the CBP60 family in the plant kingdom. Deep learning-predicted structures, cDNA/amino acid sequences, species of origin, and access to external"
- [methods] Downstream structural prediction from curated sequences: "This database features more than 2,000 proteins of the CBP60 family in the plant kingdom."
