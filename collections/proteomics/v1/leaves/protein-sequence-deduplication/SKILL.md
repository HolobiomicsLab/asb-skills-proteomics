---
name: protein-sequence-deduplication
description: Use when a positive/negative protein sequence dataset (e.g., R protein
  vs non-R protein FASTA) must be redundancy-reduced before feature embedding and
  classifier training, to prevent near-identical homologs from inflating cross-validation
  performance.
when_to_use_negative:
- Input sequences are already non-redundant or the dataset was built specifically
  to include near-homologous variants for evaluation
- The downstream analysis does not involve sequence-based feature extraction or classifier
  training, so redundancy is not a source of bias (e.g., simple annotation counting)
- Working with nucleotide or other non-protein sequences without adjusting CD-HIT
  settings for the sequence type
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: CD-HIT
  role: Clusters sequences at a 30% identity cutoff and removes redundant sequences
    from the positive and negative FASTA sets, retaining cluster representatives
- name: prPred-DRLF
  role: Source toolkit whose GitHub dataset directory provides the Wang et al. [9]
    plant R protein dataset to be filtered
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:e124e9b51ad54df9fb78a093228a38ff06ed687944c4a906aaa5f0890a524fa5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - CD-HIT
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# protein-sequence-deduplication

## Summary

Deduplicate protein FASTA datasets by clustering redundant sequences with CD-HIT at a 30% identity cutoff before feature extraction and model construction, as done for the plant R protein dataset (Wang et al. [9]) used by prPred-DRLF.

## When to use

Use when a positive/negative protein sequence dataset (e.g., R protein vs non-R protein FASTA) must be redundancy-reduced before feature embedding and classifier training, to prevent near-identical homologs from inflating cross-validation performance.

## When NOT to use

- Input sequences are already non-redundant or the dataset was built specifically to include near-homologous variants for evaluation
- The downstream analysis does not involve sequence-based feature extraction or classifier training, so redundancy is not a source of bias (e.g., simple annotation counting)
- Working with nucleotide or other non-protein sequences without adjusting CD-HIT settings for the sequence type

## Inputs

- Positive-class protein sequences in FASTA format (plant R proteins from Wang et al. [9], obtained from the prPred-DRLF GitHub dataset directory)
- Negative-class protein sequences in FASTA format (non-R proteins)

## Outputs

- Deduplicated FASTA file (cdhit_filtered.fasta) with cluster representative sequences
- Deduplicated dataset counts: 456 protein sequences (152 positive, 304 negative)
- Train/test partition sizes derived from the filtered dataset (364 training, 92 independent test samples)

## How to apply

Obtain the source dataset (the plant R protein dataset of Wang et al. [9] from the prPred-DRLF GitHub repository dataset directory) as positive and negative FASTA files. Run CD-HIT separately on the positive (R protein) and negative (non-R protein) FASTA files with the sequence identity threshold set to 0.3 (30% identity), which clusters sequences sharing high similarity and discards all but one representative per cluster. Retain the cluster representatives as the deduplicated dataset and save them to a FASTA file (e.g., cdhit_filtered.fasta). The rationale is that redundant homologous sequences would bias downstream feature extraction (BiLSTM, UniRep, TAPE-BERT via eFeature) and LGBM model training. Validate that the output FASTA contains exactly 456 sequences — 152 positive and 304 negative for this dataset — which are then split into 364 training and 92 independent test samples.

## Related tools

- **CD-HIT** (Clusters sequences at a 30% identity cutoff and removes redundant sequences from the positive and negative FASTA sets, retaining cluster representatives)
- **prPred-DRLF** (Source toolkit whose GitHub dataset directory provides the Wang et al. [9] plant R protein dataset to be filtered) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
cd-hit -i positive.fasta -o cdhit_filtered_positive.fasta -c 0.3 ; cd-hit -i negative.fasta -o cdhit_filtered_negative.fasta -c 0.3
```

## Evaluation signals

- Output FASTA must contain exactly 456 sequences for this dataset (152 positive, 304 negative) after CD-HIT filtering at the 30% identity cutoff
- Sequence counts in the filtered FASTA match the numbers reported in the article's methods section
- The filtered dataset partitions into 364 training samples and 92 independent test samples without sequence overlap
- Each retained sequence is a CD-HIT cluster representative (no two retained sequences exceed the 0.3 identity threshold)
- Downstream feature embedding (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D) runs on the deduplicated sequences without duplicate entries

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources, and parallel computing is not yet incorporated into the program
- A 30% identity cutoff is aggressive; very low cutoffs may merge distantly related homologs and reduce dataset size more than intended
- The specific counts (456 sequences; 152 positive, 304 negative) are dataset-specific to Wang et al. [9] and will differ for other inputs

## Evidence

- [methods] rationale for dataset preparation: [evidence span withheld — anchor status: not_found]
