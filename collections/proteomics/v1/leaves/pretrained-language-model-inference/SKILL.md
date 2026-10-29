---
name: pretrained-language-model-inference
description: Use when raw protein sequences in FASTA format must be converted to fixed-length
  numerical feature vectors for machine learning classification, and traditional hand-crafted
  features are insufficient.
when_to_use_negative:
- Input is already a numerical feature table or embedding matrix — no sequence encoding
  is needed
- Only short motif/domain-level classification is required and HMMER-style profile
  matching suffices (note HMMER was evaluated on positives only because most negatives
  were annotated incompletely)
- Compute resources are severely limited — the authors state deep representation learning
  methods 'usually require an elevated level of computing resources' and parallel
  computing is not yet incorporated
edam_operation: http://edamontology.org/operation_3094
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: eFeature
  role: Web/online interface through which the pretrained BiLSTM, UniRep, and TAPE-BERT
    models are run to embed protein sequences into numerical vectors
  repo: http://lab.malab.cn/soft/eFeature/index.html
- name: BiLSTM
  role: Pretrained bidirectional long short-term memory model producing 3605-dimensional
    per-sequence embeddings
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred-DRLF
  role: Open-source Python 3.7 toolkit that packages the embedding pipeline, feature
    fusion, and LGBM prediction
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: TAPE (tasks assessing protein embedding)
  role: BERT-based semi-supervised protein sequence model (768D embeddings) evaluated
    as an alternative embedding source
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:233b3858f51154f062d2e9a4217d36f8821cd568e8662e6fc26824768a930471
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiLSTM
  - TAPE (tasks assessing protein embedding)
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_3094
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Pretrained language model inference

## Summary

Encodes raw protein amino acid sequences into dense numerical vectors by running pretrained deep representation learning models (BiLSTM, UniRep, TAPE-BERT) through eFeature, then fusing embeddings (e.g., BiLSTM 3605D + UniRep 1900D = 5505D) as input features for downstream classifiers such as LGBM.

## When to use

Use when raw protein sequences in FASTA format must be converted to fixed-length numerical feature vectors for machine learning classification, and traditional hand-crafted features are insufficient.

## When NOT to use

- Input is already a numerical feature table or embedding matrix — no sequence encoding is needed
- Only short motif/domain-level classification is required and HMMER-style profile matching suffices (note HMMER was evaluated on positives only because most negatives were annotated incompletely)
- Compute resources are severely limited — the authors state deep representation learning methods 'usually require an elevated level of computing resources' and parallel computing is not yet incorporated

## Inputs

- Protein sequences in FASTA format (e.g., 456 CD-HIT-filtered plant R protein sequences: 152 positive, 304 negative)
- Pretrained embedding models (BiLSTM, UniRep 1900-hidden-unit mLSTM pretrained on UniRef50, TAPE-BERT) accessed via eFeature

## Outputs

- Per-sequence numerical embedding vectors (BiLSTM 3605D, UniRep 1900D, TAPE-BERT 768D)
- Fused feature matrix (e.g., fused_embeddings.csv with 5505 feature columns plus sequence labels 1 = R protein, 0 = non-R protein)

## How to apply

Load the CD-HIT-filtered protein FASTA (e.g., the 456-sequence plant R protein dataset filtered at a 30% identity cutoff) and run each pretrained embedding model through eFeature: the BiLSTM model produces a 3605-dimensional vector per sequence, the UniRep 1900-hidden-unit mLSTM (pretrained on UniRef50) produces a 1900-dimensional vector, and TAPE-BERT produces 768 dimensions. Concatenate the chosen embeddings per sequence — here BiLSTM + UniRep into a fused 5505-dimensional feature vector — and attach labels (1 = R protein, 0 = non-R protein). The fused BiLSTM + UniRep features were found to outperform other feature combinations when fed to an LGBM classifier. Python 3.7 with torch 1.2.x and tape_proteins 0.5 from the prPred-DRLF repository provides the runtime; GPU acceleration is recommended since these deep representation models are computationally demanding.

## Related tools

- **eFeature** (Web/online interface through which the pretrained BiLSTM, UniRep, and TAPE-BERT models are run to embed protein sequences into numerical vectors) — http://lab.malab.cn/soft/eFeature/index.html
- **BiLSTM** (Pretrained bidirectional long short-term memory model producing 3605-dimensional per-sequence embeddings) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred-DRLF** (Open-source Python 3.7 toolkit that packages the embedding pipeline, feature fusion, and LGBM prediction) — https://github.com/Wangys-prog/prPred-DRLF
- **TAPE (tasks assessing protein embedding)** (BERT-based semi-supervised protein sequence model (768D embeddings) evaluated as an alternative embedding source)

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Fused feature matrix has exactly 5505 feature columns (3605 BiLSTM + 1900 UniRep) — or 768 if TAPE-BERT alone was used
- One embedding row per input sequence (e.g., 456 rows for the CD-HIT-filtered dataset)
- Labels correctly attached as 1 = R protein, 0 = non-R protein (152 positive, 304 negative in the reference dataset)
- Downstream LGBM performance on independent test reaches the reported ranges (accuracy 0.956, F1-score 0.933, AUC 0.997) when fused features are used
- Embedding script completes under the pinned environment (Python 3.7, torch 1.2.x, tape_proteins 0.5, numpy 1.19.2, pandas 1.2.0)

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources (stated limitation)
- Parallel computing is not yet incorporated into the program; the authors plan it for future studies to achieve high computational efficiency
- GPU strongly recommended by the README; CPU-only execution is supported but slower

## Evidence

- [abstract] The results show that the fused features of bidirectional long short-term memory (BiLSTM) embedding and unified representation (Uni Rep) embedding have a better performance than other features: [evidence span withheld — anchor status: not_found]
- [methods] TAPE-BERT usinge Feature (http://lab.malab.cn/soft/e Feature/index.html): [evidence span withheld — anchor status: not_found]
