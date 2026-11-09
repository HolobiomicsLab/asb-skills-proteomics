---
name: feature-vector-concatenation-fusion
description: Use when multiple per-sequence embedding vectors (e.g. BiLSTM 3605D and
  UniRep 1900D from eFeature) must be merged into one fixed-width feature matrix before
  classifier training, with exact dimensionality checks.
when_to_use_negative:
- Input is already a single fixed-width feature table — no additional embeddings to
  fuse exist
- Only one embedding model was run (e.g. TAPE-BERT 768D alone) and there is no second
  vector to concatenate
- Feature matrices have different sequence rows/IDs that cannot be aligned — concatenation
  would mislabel proteins
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: eFeature
  role: Platform used to run the pretrained BiLSTM and UniRep embedding models and
    generate the per-sequence feature vectors that are fused
- name: BiLSTM
  role: Pretrained embedding model producing the 3605D per-sequence component of the
    fused vector
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred-DRLF
  role: Toolkit whose pipeline consumes the fused 5505D features for LGBM-based plant
    R protein prediction
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:fef98971fea7dcfc041a23a0d3cec84bec6ce1a1cbf54e604eeb8d7d1f28bbc6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wangys-prog/prPred-DRLF
  tools:
  - BiLSTM
  - eFeature
  - prPred-DRLF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# feature-vector-concatenation-fusion

## Summary

Per-sequence fusion of deep representation learning protein embeddings by column-wise concatenation: in prPred-DRLF, a 3605D BiLSTM embedding and a 1900D UniRep embedding are merged into a single 5505D feature vector per protein for plant R protein identification.

## When to use

Use when multiple per-sequence embedding vectors (e.g. BiLSTM 3605D and UniRep 1900D from eFeature) must be merged into one fixed-width feature matrix before classifier training, with exact dimensionality checks.

## When NOT to use

- Input is already a single fixed-width feature table — no additional embeddings to fuse exist
- Only one embedding model was run (e.g. TAPE-BERT 768D alone) and there is no second vector to concatenate
- Feature matrices have different sequence rows/IDs that cannot be aligned — concatenation would mislabel proteins

## Inputs

- BiLSTM embedding matrix (3605D per sequence, produced via eFeature)
- UniRep embedding matrix (1900D per sequence, from 1900-hidden-unit mLSTM pretrained on UniRef50 via eFeature)
- Sequence labels (1 = R protein, 0 = non-R protein) for the 456-sequence CD-HIT-filtered dataset

## Outputs

- Fused 5505D feature matrix (fused_embeddings.csv) with one row per input sequence and sequence labels
- Fused feature matrix with exactly 5505 feature columns (3605 + 1900) and 456 rows

## How to apply

Run each pretrained embedding model through eFeature on the CD-HIT-filtered (30% identity cutoff) FASTA dataset: the pretrained BiLSTM model yields a 3605D vector per protein and the UniRep 1900-hidden-unit mLSTM (pretrained on UniRef50) yields a 1900D vector per protein. Align the two feature matrices by sequence ID and concatenate column-wise so every row is one protein, preserving labels (1 = R protein, 0 = non-R protein). Save the result as a fused feature table (fused_embeddings.csv). The rationale is empirical: the paper found the fused BiLSTM+UniRep features outperform other feature combinations, and the fused vector (3605D + 1900D = 5505D) is the input to feature selection (RF, LGBM, MRMD3.0) and LGBM classification. Validate that the fused matrix has exactly 5505 feature columns and one row per input sequence (456 sequences after CD-HIT: 152 positive, 304 negative).

## Related tools

- **eFeature** (Platform used to run the pretrained BiLSTM and UniRep embedding models and generate the per-sequence feature vectors that are fused)
- **BiLSTM** (Pretrained embedding model producing the 3605D per-sequence component of the fused vector) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred-DRLF** (Toolkit whose pipeline consumes the fused 5505D features for LGBM-based plant R protein prediction) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
import pandas as pd; fused = pd.concat([bilstm_df, unirep_df.set_index('id')], axis=1); assert fused.shape[1] == 5505; fused.to_csv('fused_embeddings.csv', index=False)
```

## Evaluation signals

- Fused feature matrix has exactly 5505 feature columns (3605 BiLSTM + 1900 UniRep)
- Fused matrix has one row per input sequence (456 rows for the CD-HIT-filtered dataset: 152 positive, 304 negative)
- Row order and sequence IDs in the fused matrix match the input FASTA and label assignment exactly
- Downstream LGBM classifier on the fused features should reproduce reported performance levels (independent test accuracy 0.956, F1 0.933, AUC 0.997)
- No NaN or dtype drift after concatenation of the two embedding blocks

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources (stated limitation of the paper)
- Parallel computing is not yet incorporated into the program; the authors plan this for future computational efficiency
- Fusion is only defined for embeddings of the same set of sequences; TAPE-BERT (768D) was evaluated separately, not fused into the 5505D vector

## Evidence

- [abstract] fused features of bidirectional long short-term memory (BiLSTM) embedding and unified representation (Uni Rep) embedding have a better performance: [evidence span withheld — anchor status: not_found]
