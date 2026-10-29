---
name: gpcr-benchmark-dataset-handling
description: Use when building or reproducing a GPCR classification benchmark from
  UniProt FASTA sequences and you must control sequence homology (CD-Hit identity
  threshold 0.8) before feature extraction or cross-validation.
when_to_use_negative:
- The input is already a homology-reduced, non-redundant feature or sequence table
  and re-clustering would discard data
- The task requires classifying individual query sequences via the pre-trained predictor
  (Predict.py) rather than reconstructing the benchmark dataset
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CD-Hit
  role: Sequence homology reduction at 0.8 identity threshold to build the non-redundant
    benchmark dataset
- name: UniProt
  role: Source of the GPCR protein sequences constituting the benchmark dataset
- name: EMCBOW-GPCR repository
  role: Provides the benchmark dataset sequences and downstream code (GetResults.py,
    Predict.py) that consume the curated FASTA
  repo: https://github.com/454170054/EMCBOW-GPCR
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:365eb9a73a6dcb38c45ef85bccb516671d95ec6629eac92eac9dc193fdc73541
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/454170054/EMCBOW-GPCR
  repo_url: https://github.com/454170054/EMCBOW-GPCR
  tools:
  - CD-Hit
  - EMCBOW-GPCR repository
  - UniProt
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gpcr-benchmark-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates the EMCBOW-GPCR GPCR benchmark dataset by downloading GPCR protein sequences from UniProt and applying CD-Hit homology reduction at a 0.8 sequence identity threshold, then serving the non-redundant FASTA to downstream feature extraction (BOW/word-embedding) and 5-fold cross-validation.

## When to use

Use when building or reproducing a GPCR classification benchmark from UniProt FASTA sequences and you must control sequence homology (CD-Hit identity threshold 0.8) before feature extraction or cross-validation.

## When NOT to use

- The input is already a homology-reduced, non-redundant feature or sequence table and re-clustering would discard data
- The task requires classifying individual query sequences via the pre-trained predictor (Predict.py) rather than reconstructing the benchmark dataset

## Inputs

- GPCR protein sequences downloaded from UniProt in FASTA format
- CD-Hit program with sequence identity threshold parameter set to 0.8

## Outputs

- benchmark_dataset.fasta: homology-reduced, non-redundant GPCR sequence set (one representative per CD-Hit cluster)
- Recorded sequence counts before and after CD-Hit clustering

## How to apply

Download the GPCR protein sequences that constitute the benchmark from UniProt as a FASTA file. Run CD-Hit on the FASTA with the sequence identity threshold set to 0.8, so sequences sharing >= 80% identity are grouped into a single cluster, and keep one representative sequence per cluster to produce the homology-reduced benchmark. Record sequence counts before and after clustering and confirm the retained set is pairwise below the 0.8 threshold per CD-Hit clustering. This homology-reduced benchmark feeds the EMCBOW-GPCR pipeline, where sequences are evaluated by '5-fold cross-validation with 20 times on benchmark dataset' and every sequence must ultimately yield a 115-dimensional BOW feature vector (reference BOW-feature performance: AUC = 0.9344).

## Related tools

- **CD-Hit** (Sequence homology reduction at 0.8 identity threshold to build the non-redundant benchmark dataset)
- **UniProt** (Source of the GPCR protein sequences constituting the benchmark dataset)
- **EMCBOW-GPCR repository** (Provides the benchmark dataset sequences and downstream code (GetResults.py, Predict.py) that consume the curated FASTA) — https://github.com/454170054/EMCBOW-GPCR

## Examples

```
cd-hit -i uniprot_gpcr.fasta -o benchmark_dataset.fasta -c 0.8
```

## Evaluation signals

- benchmark_dataset.fasta exists and parses as a valid FASTA file
- Sequence count after CD-Hit clustering (0.8 identity) is fewer than the original UniProt download
- All retained sequences are pairwise below the 0.8 sequence identity threshold per CD-Hit clustering
- Downstream, every benchmark sequence yields a well-formed 115-dimensional BOW feature vector with row count equal to the number of benchmark sequences
- Reference expectation for classifier quality on this benchmark: BOW features achieve AUC = 0.9344 (combined features AUC = 0.9423) under 5-fold cross-validation repeated 20 times

## Limitations

- Version numbers for the software packages used are not stated in the article; they must be taken from the repository requirements.txt
- The paper does not provide dataset accessions in the discussion section, so exact UniProt download provenance must be recovered from the source repository
- The README requires a Python 3.7.0 virtual environment with packages from requirements.txt, and models must be generated via GetResults.py before prediction

## Evidence

- [methods] UniProt + CD-Hit 0.8 curation: [evidence span withheld — anchor status: not_found]
