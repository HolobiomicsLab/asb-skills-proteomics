---
name: fasta-sequence-handling
description: Use when raw protein sequences downloaded from UniProt must be turned
  into a homology-reduced benchmark dataset in FASTA format before classifier training
  or cross-validation (e.g., sequences sharing >= 80% identity need to be collapsed).
when_to_use_negative:
- The input FASTA has already been de-duplicated or homology-reduced at the required
  identity threshold (e.g., output of a prior CD-Hit run)
- The task is running the trained EMCBOW-GPCR predictor on a single FASTA file (use
  Predict.py instead of re-curation)
- The sequences are not protein sequences from the GPCR benchmark context and a different
  identity cutoff or clustering granularity is required
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: UniProt
  role: Source database from which the benchmark dataset GPCR protein sequences are
    downloaded in FASTA format
- name: CD-Hit
  role: Sequence clustering tool used to reduce homology with a sequence identity
    threshold of 0.8, retaining one representative per cluster
- name: EMCBOW-GPCR (Predict.py / GetResults.py)
  role: 'Downstream consumer of the FASTA-formatted sequences: GetResults.py generates
    models and Predict.py predicts GPCRs given a FASTA file'
  repo: https://github.com/454170054/EMCBOW-GPCR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.044
    title: 'EMCBOW-GPCR: A method for identifying G-protein coupled receptors based
      on word embedding and wordbooks'
  source_content_hash: sha256:524ac073a465d5ead7f5eed2508f89a178db4ac719cc0c5f975dd456287e6dae
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
  - EMCBOW-GPCR (Predict.py / GetResults.py)
  - UniProt
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fasta-sequence-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate a non-redundant FASTA protein dataset by downloading source sequences from UniProt and clustering them with CD-Hit at a 0.8 sequence identity threshold, keeping one representative per cluster. This reproduces the homology-reduced GPCR benchmark dataset used to evaluate EMCBOW-GPCR.

## When to use

Use when raw protein sequences downloaded from UniProt must be turned into a homology-reduced benchmark dataset in FASTA format before classifier training or cross-validation (e.g., sequences sharing >= 80% identity need to be collapsed).

## When NOT to use

- The input FASTA has already been de-duplicated or homology-reduced at the required identity threshold (e.g., output of a prior CD-Hit run)
- The task is running the trained EMCBOW-GPCR predictor on a single FASTA file (use Predict.py instead of re-curation)
- The sequences are not protein sequences from the GPCR benchmark context and a different identity cutoff or clustering granularity is required

## Inputs

- UniProt FASTA file of GPCR protein sequences (original benchmark dataset download)
- CD-Hit sequence identity threshold parameter (0.8)

## Outputs

- benchmark_dataset.fasta (homology-reduced, non-redundant FASTA with one representative sequence per CD-Hit cluster)
- Recorded sequence counts before and after CD-Hit clustering

## How to apply

Download the GPCR protein sequences that constitute the benchmark dataset from UniProt as a FASTA file. Run CD-Hit on the retrieved FASTA with the sequence identity threshold set to 0.8, so sequences sharing >= 80% identity are grouped into a single cluster. Keep one representative sequence per CD-Hit cluster and write the non-redundant sequences to benchmark_dataset.fasta, recording the sequence counts before and after clustering. This reduction is what supports the paper's evaluation protocol of '5-fold cross-validation with 20 times on benchmark dataset', because it limits homology-driven inflation of accuracy. Validate that the output is a valid FASTA file containing fewer sequences than the original UniProt download and that all retained sequences are pairwise below the 0.8 identity threshold per CD-Hit clustering.

## Related tools

- **UniProt** (Source database from which the benchmark dataset GPCR protein sequences are downloaded in FASTA format)
- **CD-Hit** (Sequence clustering tool used to reduce homology with a sequence identity threshold of 0.8, retaining one representative per cluster)
- **EMCBOW-GPCR (Predict.py / GetResults.py)** (Downstream consumer of the FASTA-formatted sequences: GetResults.py generates models and Predict.py predicts GPCRs given a FASTA file) — https://github.com/454170054/EMCBOW-GPCR

## Examples

```
cd-hit -i uniprot_gpcr.fasta -o benchmark_dataset.fasta -c 0.8
```

## Evaluation signals

- benchmark_dataset.fasta exists and parses as a valid FASTA file
- The number of sequences in benchmark_dataset.fasta is strictly smaller than the number in the original UniProt download
- All retained sequences are pairwise below the 0.8 identity threshold according to CD-Hit clustering
- Sequence counts before and after clustering are recorded and consistent with one representative per cluster
- The homology-reduced dataset is the one evaluated via '5-fold cross-validation with 20 times on benchmark dataset' in the paper's protocol

## Limitations

- The 0.8 identity threshold is dataset-specific; different homology cutoffs would change the benchmark composition and reported metrics
- Keeping one representative per CD-Hit cluster discards within-cluster sequence diversity, which may affect downstream feature extraction (e.g., CBOW word-embedding quality depends on the hyperparameter N)
- Software version numbers for tools used in the pipeline are not stated in the paper; they must be recovered from the repository's requirements.txt

## Evidence

- [methods] The benchmark dataset sequences were downloaded from the UniProt database, and the CD-Hit program was used to reduce sequence homology with a sequence identity threshold of 0.8: [evidence span withheld — anchor status: not_found]
