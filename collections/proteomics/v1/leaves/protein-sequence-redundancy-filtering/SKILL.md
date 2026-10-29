---
name: protein-sequence-redundancy-filtering
description: Use when preparing a raw protein FASTA dataset (e.g., Wang et al. plant
  R protein sequences) for classifier training and you must remove homologous redundancy
  to prevent overestimated cross-validation and independent-test performance.
when_to_use_negative:
- The input is already a curated, non-redundant benchmark dataset (e.g., a prefiltered
  independent test set like the 92-sample test_data.fasta) — re-filtering could alter
  the reported benchmark composition.
- The downstream model needs full sequence diversity of very small datasets, where
  a 30% cutoff might discard too many positives (e.g., only 152 R proteins remain
  in prPred-DRLF).
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: CD-HIT
  role: Clusters the input protein FASTA at a 30% identity cutoff and removes redundant
    sequences prior to train/test splitting and feature encoding
- name: prPred-DRLF
  role: Downstream pipeline whose dataset was produced by this CD-HIT filtering step;
    provides the dataset download and the independent test FASTA
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:68ee5c039a4208374f86d521cad3fd3f91fb1ca2b134d97095a2c396c2928371
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - CD-HIT
  - Python
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-sequence-redundancy-filtering

## Summary

Removes redundant protein sequences from a FASTA dataset using CD-HIT with a 30% identity cutoff before splitting into training/independent-test sets, as done in the prPred-DRLF plant R protein pipeline.

## When to use

Use when preparing a raw protein FASTA dataset (e.g., Wang et al. plant R protein sequences) for classifier training and you must remove homologous redundancy to prevent overestimated cross-validation and independent-test performance.

## When NOT to use

- The input is already a curated, non-redundant benchmark dataset (e.g., a prefiltered independent test set like the 92-sample test_data.fasta) — re-filtering could alter the reported benchmark composition.
- The downstream model needs full sequence diversity of very small datasets, where a 30% cutoff might discard too many positives (e.g., only 152 R proteins remain in prPred-DRLF).

## Inputs

- Raw protein sequences in FASTA format (plant R protein and non-R protein sequences from the Wang et al. dataset)

## Outputs

- Redundancy-filtered FASTA containing 456 sequences (152 positive, 304 negative)
- Split sample lists: 364 training samples and 92 independent test samples with binary labels (1 = R protein, 0 = non-R protein)

## How to apply

Run CD-HIT on the input FASTA with the identity threshold set to 0.30, clustering sequences at or above 30% sequence identity so only cluster representatives are retained. In prPred-DRLF this step reduced the Wang et al. plant R protein dataset to 456 sequences (152 positive, 304 negative), which were then split into 364 training and 92 independent-test samples. Perform redundancy removal BEFORE any train/test split so homologs cannot leak between the training set and the independent test set. After filtering, verify the class balance (positives vs negatives) and encode the retained sequences into numerical vectors (e.g., BiLSTM 3605D, UniRep 1900D) downstream. The 30% cutoff is deliberately stringent, well below typical 0.7–0.9 thresholds, to maximize dissimilarity between clusters.

## Related tools

- **CD-HIT** (Clusters the input protein FASTA at a 30% identity cutoff and removes redundant sequences prior to train/test splitting and feature encoding)
- **prPred-DRLF** (Downstream pipeline whose dataset was produced by this CD-HIT filtering step; provides the dataset download and the independent test FASTA) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
cd-hit -i plant_R_proteins.fasta -o plant_R_proteins_cdhit30.fasta -c 0.30
```

## Evaluation signals

- Post-filter dataset size matches 456 sequences: 152 positive and 304 negative samples.
- Resulting split contains 364 training samples and 92 independent test samples.
- No pair of sequences across the training/independent-test split exceeds 30% sequence identity (no homolog leakage).
- Downstream metrics computed on this filtered dataset reproduce the reported independent-test performance (accuracy 0.956, F1-score 0.933, AUC 0.997) within tolerance.

## Limitations

- A stringent 30% identity cutoff can substantially shrink small positive classes (only 152 R protein sequences remained), limiting training data for deep encoders.
- Deep representation learning feature methods require an elevated level of computing resources, and prPred-DRLF does not yet incorporate parallel computing, which slows processing of filtered datasets.

## Evidence
