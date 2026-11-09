---
name: sequence-homology-reduction
description: Use when building a protein benchmark dataset from raw database downloads
  (e.g. UniProt FASTA) where redundant homologous sequences must be removed before
  training or cross-validating a classifier such as EMCBOW-GPCR.
when_to_use_negative:
- The input is already a homology-reduced or non-redundant sequence set (running CD-Hit
  again at 0.8 would be redundant)
- The task is classifier training/inference on an existing benchmark rather than benchmark
  dataset construction
- The sequences are nucleotide rather than protein (protein CD-Hit parameters would
  not apply directly)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: UniProt
  role: Source database from which the GPCR benchmark protein sequences are downloaded
    in FASTA format
- name: CD-Hit
  role: Sequence clustering program used to reduce homology with a 0.8 sequence identity
    threshold, yielding one representative per cluster
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:ed31417bf0f4ec8c82f102c04b75be9ffb658c95392c08ef2c200222d2615b04
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - CD-Hit
  - UniProt
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# sequence-homology-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a non-redundant protein benchmark dataset by clustering UniProt GPCR sequences with CD-Hit at a 0.8 sequence identity threshold and keeping one representative per cluster, as done to prepare the EMCBOW-GPCR benchmark evaluated by 5-fold cross-validation with 20 repeats.

## When to use

Use when building a protein benchmark dataset from raw database downloads (e.g. UniProt FASTA) where redundant homologous sequences must be removed before training or cross-validating a classifier such as EMCBOW-GPCR.

## When NOT to use

- The input is already a homology-reduced or non-redundant sequence set (running CD-Hit again at 0.8 would be redundant)
- The task is classifier training/inference on an existing benchmark rather than benchmark dataset construction
- The sequences are nucleotide rather than protein (protein CD-Hit parameters would not apply directly)

## Inputs

- FASTA file of GPCR protein sequences downloaded from UniProt (raw benchmark dataset sequences)

## Outputs

- benchmark_dataset.fasta — non-redundant FASTA containing one representative sequence per CD-Hit cluster
- Sequence counts before and after homology reduction

## How to apply

Download the GPCR protein sequences that constitute the benchmark dataset from the UniProt database as a FASTA file. Run CD-Hit on the retrieved FASTA with the sequence identity threshold (-c) set to 0.8, so that sequences sharing >= 80% identity are grouped into a single cluster. Keep one representative sequence per CD-Hit cluster to produce the homology-reduced dataset, and write the non-redundant sequences to a file such as benchmark_dataset.fasta while recording sequence counts before and after clustering. This homology reduction precedes feature extraction (BOW, word embedding via Gensim CBOW models) and XGBoost-based classification, and supports the paper's evaluation protocol of 5-fold cross-validation repeated 20 times on the benchmark dataset.

## Related tools

- **UniProt** (Source database from which the GPCR benchmark protein sequences are downloaded in FASTA format)
- **CD-Hit** (Sequence clustering program used to reduce homology with a 0.8 sequence identity threshold, yielding one representative per cluster)

## Examples

```
cd-hit -i uniprot_gpcr.fasta -o benchmark_dataset.fasta -c 0.8
```

## Evaluation signals

- benchmark_dataset.fasta exists and is a valid FASTA file
- Number of sequences in benchmark_dataset.fasta is fewer than in the original UniProt download
- All retained sequences are pairwise identical below the 0.8 identity threshold according to CD-Hit clustering
- Sequence counts before and after clustering are recorded and consistent with the clustering result
- The reduced dataset supports the paper's evaluation protocol of 5-fold cross-validation with 20 repetitions on the benchmark dataset

## Limitations

- The homology-reduction threshold is fixed at 0.8 sequence identity in this workflow; a different threshold may be needed for other benchmarks and could change dataset size and classifier performance
- Version numbers for the software tools are not stated in the paper (only Python package versions are available via the repo's requirements.txt, with Python 3.7.0 specified in the README)
- Keeping only one representative per CD-Hit cluster discards clustered homologs, which may reduce sequence diversity available for training

## Evidence
