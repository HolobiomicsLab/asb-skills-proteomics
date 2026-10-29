---
name: transcript-protein-count-verification
description: Use when reproducing or auditing a curated gene family database and you
  must confirm that homolog, transcript, and protein counts match published totals
  before downstream structural or phylogenetic analyses.
when_to_use_negative:
- The database counts are not in question and you only need to run structure prediction
  or clustering on existing CBP60-DB entries (use the CBP60-DB web app or ColabFold
  notebook instead).
- Your study focuses on functional annotation of CBP60g/SARD1 paralogs rather than
  verifying the sequence collection inventory.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0780
tools:
- name: Gramene
  role: Comparative genomics website queried to identify CBP60 gene homologs and download
    gene, cDNA/transcript, and protein sequences across plant species with published
    genomes
  repo: https://www.gramene.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pld3.509
    title: 'CBP60‐DB: An AlphaFold‐predicted plant kingdom‐wide database of the CALMODULIN‐BINDING
      PROTEIN 60 protein family with a novel structural clustering algorithm'
  source_content_hash: sha256:96a3d2c8f5bafda603779caf246d6727a915e491d49d5573530cf9d7a852f670
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/KeaunAmani/cbp60db
  tools:
  - Gramene
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# transcript-protein-count-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A verification skill for reproducing the sequence inventory of a gene family database: tallying unique gene homologs, cDNA/transcripts, and proteins across genomes and comparing each count against reported values (here 1052 genes, 2376 transcripts, 1996 proteins across 62 plant genomes for CBP60-DB).

## When to use

Use when reproducing or auditing a curated gene family database and you must confirm that homolog, transcript, and protein counts match published totals before downstream structural or phylogenetic analyses.

## When NOT to use

- The database counts are not in question and you only need to run structure prediction or clustering on existing CBP60-DB entries (use the CBP60-DB web app or ColabFold notebook instead).
- Your study focuses on functional annotation of CBP60g/SARD1 paralogs rather than verifying the sequence collection inventory.

## Inputs

- Gramene comparative genomics query results for CBP60 gene homologs (https://www.gramene.org)
- Downloaded gene, cDNA/transcript, and protein sequence records (e.g., FASTA) per homolog per species
- List of plant species with published genomes

## Outputs

- cbp60_sequence_counts.csv with verified tallies: 1052 gene homologs, 2376 unique transcripts, 1996 unique proteins, 62 genomes
- Deduplicated transcript and protein sequence sets ready for structure prediction (CBP60-DB entries)

## How to apply

Query a comparative genomics resource (here the Gramene website, https://www.gramene.org) to identify gene homologs across phylogenetically diverse species with published genomes, then download the gene, cDNA/transcript, and protein sequences for each homolog. Deduplicate transcripts and protein sequences so each category contains only unique entries, since one gene can encode multiple transcripts and multiple transcripts can map to the same protein (e.g., 1052 CBP60 genes encoding 2376 unique transcripts and 1996 unique proteins). Tally totals per category and record the number of source genomes (62 for CBP60-DB). Write the verified counts to a machine-readable file such as cbp60_sequence_counts.csv and compare each value against the published figures, treating any mismatch as a reproduction failure that must be resolved before the dataset is used for AlphaFold2/ColabFold structure prediction or clustering.

## Related tools

- **Gramene** (Comparative genomics website queried to identify CBP60 gene homologs and download gene, cDNA/transcript, and protein sequences across plant species with published genomes) — https://www.gramene.org

## Examples

```
python verify_counts.py --source gramene --dedupe transcripts,proteins --expected genes=1052,transcripts=2376,proteins=1996,genomes=62 --out cbp60_sequence_counts.csv
```

## Evaluation signals

- Gene homolog count equals 1052 after Gramene query and deduplication
- Unique cDNA/transcript count equals 2376
- Unique protein count equals 1996 (the CBP60-DB web app describes 'more than 2,000 proteins of the CBP60 family', consistent with 1996)
- Number of source plant genomes with published assemblies equals 62
- Counts are written to cbp60_sequence_counts.csv with one verified value per category

## Limitations

- Counts depend on the set of species with published genomes available in Gramene at query time; newly released genomes will change the tallies.
- Deduplication criteria (what counts as a 'unique' transcript or protein) must match the original study's definition or counts will not be comparable.
- The article does not publish the per-species breakdown, so only the four aggregate totals can be verified.

## Evidence

- [other] cbp60_sequence_counts.csv workflow: [evidence span withheld — anchor status: not_found]
- [other] Gramene as sequence source: [evidence span withheld — anchor status: not_found]
