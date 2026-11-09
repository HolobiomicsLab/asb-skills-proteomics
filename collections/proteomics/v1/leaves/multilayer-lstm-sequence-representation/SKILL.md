---
name: multilayer-lstm-sequence-representation
description: Use when raw protein sequences in FASTA format must be converted to numerical
  feature vectors for ML classification, e.g. fusing BiLSTM (3605D) and UniRep (1900D)
  embeddings into a 5505D matrix.
when_to_use_negative:
- Input is already a numerical feature table or embedding matrix — this skill operates
  on raw amino acid sequences
- A lightweight traditional feature representation is required (e.g. k-spaced amino
  acid group pairs as in the original prPred), since deep representation learning
  features require an elevated level of computing resources
- Only TAPE-BERT (768D) embeddings are needed, as the paper reports the fused BiLSTM
  + UniRep vector — not TAPE-BERT alone — performs best
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: eFeature
  role: Web/tool interface through which the pretrained BiLSTM and UniRep models are
    invoked to embed protein sequences into numerical vectors
  repo: http://lab.malab.cn/soft/eFeature/index.html
- name: BiLSTM
  role: Pretrained bidirectional long short-term memory model that yields the 3605D
    per-sequence embedding component
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: UniRep
  role: 1900-hidden-unit mLSTM pretrained on UniRef50 that yields the 1900D per-sequence
    embedding component
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: prPred-DRLF
  role: Open-source Python 3.7 toolkit that packages the embedding pipeline and LGBM
    prediction over the fused features
  repo: https://github.com/Wangys-prog/prPred-DRLF
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:e443203e8656195cf82ddb080e7945747c81d62c59f93993333396b29fa16305
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiLSTM
  - UniRep
  - eFeature
  - prPred-DRLF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multilayer-lstm-sequence-representation

## Summary

Encodes raw protein amino acid sequences into fixed-length numerical vectors by running pretrained deep representation learning models (BiLSTM and UniRep) through eFeature and fusing their embeddings into a single 5505-dimensional feature vector for downstream plant R protein classification with an LGBM classifier.

## When to use

Use when raw protein sequences in FASTA format must be converted to numerical feature vectors for ML classification, e.g. fusing BiLSTM (3605D) and UniRep (1900D) embeddings into a 5505D matrix.

## When NOT to use

- Input is already a numerical feature table or embedding matrix — this skill operates on raw amino acid sequences
- A lightweight traditional feature representation is required (e.g. k-spaced amino acid group pairs as in the original prPred), since deep representation learning features require an elevated level of computing resources
- Only TAPE-BERT (768D) embeddings are needed, as the paper reports the fused BiLSTM + UniRep vector — not TAPE-BERT alone — performs best

## Inputs

- Protein amino acid sequences in FASTA format (e.g. the CD-HIT-filtered plant R protein dataset of 456 sequences from Wang et al. [9])
- Pretrained BiLSTM embedding model accessed via eFeature
- Pretrained UniRep model (1900-hidden-unit mLSTM pretrained on UniRef50) accessed via eFeature

## Outputs

- Per-sequence BiLSTM embedding matrix (3605 features per sequence)
- Per-sequence UniRep embedding matrix (1900 features per sequence)
- Fused 5505-dimensional feature matrix (3605 + 1900) with binary sequence labels (1 = R protein, 0 = non-R protein), saved as fused_embeddings.csv

## How to apply

Load the CD-HIT-filtered dataset (30% identity cutoff; the prPred-DRLF dataset has 456 sequences: 152 positive R proteins, 304 negative) in FASTA format from the prPred-DRLF GitHub repository. Run the pretrained BiLSTM embedding model through eFeature to produce a 3605D vector per sequence, then run the pretrained UniRep model — a 1900-hidden-unit mLSTM pretrained on UniRef50 — to produce a 1900D vector per sequence. Concatenate the two embeddings per sequence into the fused 5505D feature vector (3605 + 1900), attaching binary labels (1 = R protein, 0 = non-R protein), and save as fused_embeddings.csv. This fused vector is the preferred representation because the paper found BiLSTM + UniRep fusion outperformed other feature combinations, including TAPE-BERT alone (768D), when fed to the LightGBM (LGBM) classifier. Operate in a Python 3.7 environment with the pinned dependencies (torch 1.2.0, numpy 1.19.2, pandas 1.2.0, tape_proteins 0.5); GPU execution is faster.

## Related tools

- **eFeature** (Web/tool interface through which the pretrained BiLSTM and UniRep models are invoked to embed protein sequences into numerical vectors) — http://lab.malab.cn/soft/eFeature/index.html
- **BiLSTM** (Pretrained bidirectional long short-term memory model that yields the 3605D per-sequence embedding component) — https://github.com/Wangys-prog/prPred-DRLF
- **UniRep** (1900-hidden-unit mLSTM pretrained on UniRef50 that yields the 1900D per-sequence embedding component) — https://github.com/Wangys-prog/prPred-DRLF
- **prPred-DRLF** (Open-source Python 3.7 toolkit that packages the embedding pipeline and LGBM prediction over the fused features) — https://github.com/Wangys-prog/prPred-DRLF

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Verify the fused feature matrix has exactly 5505 feature columns (3605 BiLSTM + 1900 UniRep) as specified by the paper's feature dimensionality
- Verify one row per input sequence — 456 rows for the CD-HIT-filtered dataset (152 positive, 304 negative)
- Confirm labels are binary (1 = R protein, 0 = non-R protein) and aligned row-wise with the FASTA input order
- Downstream LGBM validation: independent test accuracy ~0.956, F1-score ~0.933, AUC ~0.997; fivefold cross-validation with precision, recall, ACC, F1-score
- Environment check: Python 3.7 with pinned versions (joblib 1.0.1, tape_proteins 0.5, numpy 1.19.2, pandas 1.2.0, Bio 0.4.1, lightgbm 2.3.0, torch 1.2.0)

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources (stated limitation in the discussion)
- Parallel computing is not yet incorporated into the program; the authors plan it for future studies to achieve high computational efficiency
- Pinpointed legacy dependencies (Python 3.7, torch 1.2.0, numpy 1.19.2, lightgbm 2.3.0) may conflict with modern environments; a GitLab mirror is provided if the model errors out

## Evidence

- [abstract] Fusion performance: [evidence span withheld — anchor status: not_found]
