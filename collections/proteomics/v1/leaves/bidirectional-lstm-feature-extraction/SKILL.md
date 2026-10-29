---
name: bidirectional-lstm-feature-extraction
description: Use when raw protein sequences in FASTA format must be converted into
  dense numerical embeddings for a machine-learning classifier (e.g. identifying plant
  R proteins with LGBM), and pretrained deep representation learning models are available.
when_to_use_negative:
- Input is already a numerical feature table — this skill operates on raw FASTA sequences
- Sequences have not been redundancy-reduced; apply CD-HIT with a 30% identity cutoff
  first as in the source workflow
- Traditional hand-crafted features (e.g. k-spaced amino acid pairs as in prPred)
  are explicitly preferred or computational resources (GPU) are unavailable, since
  deep representation learning features require elevated computing resources
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: eFeature
  role: Web/tool interface through which the pretrained BiLSTM model embeds each protein
    sequence into its 3605D numerical vector
  repo: http://lab.malab.cn/soft/eFeature/index.html
- name: prPred-DRLF
  role: Toolkit that packages the BiLSTM embedding, feature fusion with UniRep, and
    LGBM prediction; provides the dataset and Python script
  repo: https://github.com/Wangys-prog/prPred-DRLF
- name: LightGBM (LGBM)
  role: Downstream classifier that consumes the fused BiLSTM+UniRep 5505D features
    for plant R protein identification
- name: CD-HIT
  role: Upstream redundancy removal (30% identity cutoff) applied before embedding
    the 456-sequence dataset
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100161
    title: 'prPred‐DRLF: Plant R protein predictor using deep representation learning
      features'
  source_content_hash: sha256:cdd6fb2bae9fa810e18c8e464313ec6b4289d51602608011ce0e0e0fb550405a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiLSTM
  - CD-HIT
  - LightGBM (LGBM)
  - eFeature
  - prPred-DRLF
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# bidirectional-lstm-feature-extraction

## Summary

Encodes raw protein amino acid sequences into fixed-length numerical feature vectors using a pretrained BiLSTM deep representation learning embedding (3605D via eFeature), which in prPred-DRLF is fused with UniRep (1900D) embeddings to give a 5505-D input for plant R protein classification with an LGBM classifier.

## When to use

Use when raw protein sequences in FASTA format must be converted into dense numerical embeddings for a machine-learning classifier (e.g. identifying plant R proteins with LGBM), and pretrained deep representation learning models are available.

## When NOT to use

- Input is already a numerical feature table — this skill operates on raw FASTA sequences
- Sequences have not been redundancy-reduced; apply CD-HIT with a 30% identity cutoff first as in the source workflow
- Traditional hand-crafted features (e.g. k-spaced amino acid pairs as in prPred) are explicitly preferred or computational resources (GPU) are unavailable, since deep representation learning features require elevated computing resources

## Inputs

- Protein amino acid sequences in FASTA format (e.g. CD-HIT-filtered plant R protein dataset, 456 sequences)
- Pretrained BiLSTM embedding model accessible through eFeature
- Sequence class labels (1 = R protein, 0 = non-R protein) for labeled datasets

## Outputs

- 3605-dimensional BiLSTM embedding vector per protein sequence
- Fused 5505-dimensional feature matrix (3605D BiLSTM + 1900D UniRep) with per-sequence labels (e.g. fused_embeddings.csv)

## How to apply

Load the CD-HIT-filtered protein FASTA (the prPred-DRLF dataset has 456 sequences after filtering with a 30% identity cutoff) and run the pretrained BiLSTM embedding model through eFeature (http://lab.malab.cn/soft/eFeature/index.html) to obtain a 3605-dimensional vector per sequence. In the full prPred-DRLF workflow, concatenate this with the 1900-dimensional UniRep embedding (1900-hidden-unit mLSTM pretrained on UniRef50) to form the fused 5505-D feature vector (3605D + 1900D). Attach class labels (1 = R protein, 0 = non-R protein) and save the feature matrix (e.g. fused_embeddings.csv) for downstream feature selection (RF, LGBM, MRMD3.0) and LGBM classification. The BiLSTM embedding is one of three representation learning models evaluated (BiLSTM, UniRep, TAPE-BERT); the fused BiLSTM+UniRep combination outperformed the others. Running the toolkit requires a Python 3.7 environment with tape_proteins==0.5, torch, and related dependencies, and is faster on GPU.

## Related tools

- **eFeature** (Web/tool interface through which the pretrained BiLSTM model embeds each protein sequence into its 3605D numerical vector) — http://lab.malab.cn/soft/eFeature/index.html
- **prPred-DRLF** (Toolkit that packages the BiLSTM embedding, feature fusion with UniRep, and LGBM prediction; provides the dataset and Python script) — https://github.com/Wangys-prog/prPred-DRLF
- **LightGBM (LGBM)** (Downstream classifier that consumes the fused BiLSTM+UniRep 5505D features for plant R protein identification)
- **CD-HIT** (Upstream redundancy removal (30% identity cutoff) applied before embedding the 456-sequence dataset)

## Examples

```
python3 prPred-DRLF.py -i ./dataset/test_data.fasta -o ./dataset/predict_result
```

## Evaluation signals

- Feature matrix has exactly 5505 columns (3605 BiLSTM + 1900 UniRep) and one row per input sequence (456 for the prPred-DRLF dataset)
- Each per-sequence vector is a dense numerical vector of the expected dimensionality (3605D for BiLSTM alone)
- Downstream LGBM model on the fused features should reproduce paper-level performance (independent test accuracy 0.956, F1-score 0.933, AUC 0.997)
- Fused BiLSTM+UniRep features should outperform single-model or TAPE-BERT (768D) features in cross-validation
- Pipeline runs successfully in the specified Python 3.7 environment (tape_proteins==0.5, torch, pandas==1.2.0, lightgbm-2.3.0)

## Limitations

- Deep representation learning feature methods require an elevated level of computing resources; GPU recommended by the README
- Parallel computing is not incorporated into the program (noted as planned future work for computational efficiency)
- Feature vectors were evaluated in the plant R protein context; transfer to other protein classification tasks is not established here

## Evidence

- [abstract] fused features of bidirectional long short-term memory (BiLSTM) embedding and unified representation (Uni Rep) embedding have a better performance than other features: [evidence span withheld — anchor status: not_found]
- [methods] The obtained multidimensional feature vectors include 3605D for BiLSTM, 1900D for Uni Rep, and 768D for TAPE-BERT: "The obtained multidimensional feature vectors include 3605D for BiLSTM, 1900D for Uni Rep, and 768D for TAPE-BERT"
- [methods] three protein representa- tion learning models, including BiLSTM, Uni Rep, and TAPE-BERT: "The protein sequences are embedded into numerical vectors by three protein representa- tion learning models, including BiLSTM, Uni Rep, and TAPE-BERT"
- [readme] prPred-DRLF is an open-source Python-based toolkit, which operates depending on the Python environment (##Python Version 3.7).: "prPred-DRLF is an open-source Python-based toolkit, which operates depending on the Python environment (##Python Version 3.7)."
- [methods] After removing the redundant sequences using CD-HIT[27]witha30%identitycutoff: "After removing the redundant sequences using CD-HIT[27]witha30%identitycutoff"
