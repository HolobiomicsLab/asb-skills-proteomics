---
name: protein-sequence-fasta-batch-processing
description: Use when a whole-proteome or large multi-protein FASTA (e.g. Z. mays,
  T. aestivum, A. sativa, H. vulgare) must be converted into per-residue plus protein-average
  ProtT5 embeddings as input to downstream phosphorylation site prediction.
when_to_use_negative:
- Input proteins contain missing or non-canonical amino-acid sequences — these must
  be discarded before embedding
- The embedding pickle already exists and the task is labeling or model training rather
  than embedding generation
- Only a handful of short sequences are needed and a single GPU run of the unmodified
  embed_ProtT5.ipynb notebook is sufficient — batch SLURM infrastructure is unnecessary
  overhead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: ProtT5-XL-U50
  role: Pretrained protein language model used to compute per-residue and protein-wise
    average embedding vectors
  repo: https://github.com/agemagician/ProtTrans
- name: PhosBoost
  role: Consumer of the generated embedding pickles for phosphorylation prediction
    training and prediction
  repo: https://github.com/eporetsky/PhosBoost
- name: ProtTrans
  role: Source repository of the embed_ProtT5.ipynb notebook modified for direct-to-pickle
    embedding generation
  repo: https://github.com/agemagician/ProtTrans
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:2102f93890d5f694728da5b95e102429bc99a05c3fdf804665cc1ffe718091dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/agemagician/ProtTrans
  tools:
  - CatBoost
  - PhosBoost
  - ProtT5-XL-U50
  - ProtTrans
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# protein-sequence-fasta-batch-processing

## Summary

Generate ProtT5-XL-U50 per-residue (Ser/Thr/Tyr) and protein-wise average embedding vectors from whole-proteome FASTA files using a modified ProtTrans embed pipeline run in SLURM batches, producing compressed pickle files ready for PhosBoost training or prediction.

## When to use

Use when a whole-proteome or large multi-protein FASTA (e.g. Z. mays, T. aestivum, A. sativa, H. vulgare) must be converted into per-residue plus protein-average ProtT5 embeddings as input to downstream phosphorylation site prediction.

## When NOT to use

- Input proteins contain missing or non-canonical amino-acid sequences — these must be discarded before embedding
- The embedding pickle already exists and the task is labeling or model training rather than embedding generation
- Only a handful of short sequences are needed and a single GPU run of the unmodified embed_ProtT5.ipynb notebook is sufficient — batch SLURM infrastructure is unnecessary overhead

## Inputs

- Proteome or phosphoprotein FASTA file (e.g. Arabidopsis.qPTMplants.fasta, Z. mays / T. aestivum / A. sativa / H. vulgare proteomes) placed in data/fasta/
- Pretrained ProtT5-XL-U50 protein language model (auto-downloaded via transformers, cached via TORCH_HOME/TRANSFORMERS_CACHE)
- SLURM environment with phosboost conda environment installed from environments.yml

## Outputs

- Compressed python pickle file(s) in data/embeddings/ containing Ser/Thr/Tyr residue embedding vectors and protein-wise average embedding vectors
- Intermediate batch pickle files in data/tmp/ (merged then removed)

## How to apply

Pre-filter the FASTA: discard proteins absent from genomic databases or containing missing/non-canonical amino acids. Run the modified embed_ProtT5.ipynb pipeline from the ProtTrans repository with the pretrained ProtT5-XL-U50 model via sbatch embed.batch.sh in 30 split batches, optionally on multi-thread CPU nodes (48 threads, 350GB RAM) rather than GPUs so sequence length is not limited by RAM, then merge batch pickle outputs with embed.merge.sh and clean data/tmp/. The pipeline writes residue-level embedding vectors for every Ser/Thr/Tyr residue plus a protein-wise average embedding vector into a single compressed python pickle, skipping the h5-extraction step of the original notebook. Set TORCH_HOME and TRANSFORMERS_CACHE to a project models folder to control where the large pretrained model is stored, and use the phosboost conda environment (environments.yml). Verify the merged pickle covers all S/T/Y residues of each input protein before feeding it to PhosBoost prediction or labeling.

## Related tools

- **ProtT5-XL-U50** (Pretrained protein language model used to compute per-residue and protein-wise average embedding vectors) — https://github.com/agemagician/ProtTrans
- **PhosBoost** (Consumer of the generated embedding pickles for phosphorylation prediction training and prediction) — https://github.com/eporetsky/PhosBoost
- **ProtTrans** (Source repository of the embed_ProtT5.ipynb notebook modified for direct-to-pickle embedding generation) — https://github.com/agemagician/ProtTrans

## Examples

```
sbatch embed.batch.sh name.fasta name 30 && sbatch embed.merge.sh name && rm data/tmp/*
```

## Evaluation signals

- Merged embedding pickle covers every Ser, Thr, and Tyr residue of each protein in the input FASTA, plus one average protein embedding per protein
- All batch pickle files in data/tmp/ were merged into the output in data/embeddings/ and tmp was cleaned (rm data/tmp/*)
- Pickle loads in the phosboost environment without an h5 extraction step and is directly usable by PhosBoost training/prediction scripts
- Proteins with missing or non-canonical amino acids and proteins absent from genomic databases were excluded before embedding
- Embedding jobs completed within the SLURM batch setup (e.g. 30 batches, 48 threads, 350GB RAM nodes) without memory-length truncation of long proteins

## Limitations

- Embedding generation on CPU nodes requires large RAM (48 threads, 350GB RAM used); on limited-memory systems long proteins may be constrained, though the README notes high RAM removed length limits
- A T5-tokenizer change in transformers can raise UnboundLocalError on sentencepiece_model_pb2, requiring protobuf installation or a transformers fix; legacy=True may be needed
- The modified notebook overwrites the existing embedding file when labeling, so prior unlabeled versions are not preserved

## Evidence

- [readme] PhosBoost was trained using the combined S/T/Y residue embedding vector data and average protein embedding vector data, generated using the pretrained ProtT5-XL-U50 pLM.: "PhosBoost was trained using the combined S/T/Y residue embedding vector data and average protein embedding vector data, generated using the pretrained ProtT5-XL-U50 pLM."
- [readme] CPU batch rationale: "While it is possible to use GPUs to generate the embedding data, we used multiple CPU nodes (48 threads, 350GB RAM) to process the input FASTA file in multiple batches."
- [readme] Modified notebook benefit: "The main benefit of this modified version is that it doesn't require the extraction of the residue and protein embedding data from the h5 files, and the generated compressed python pickle file is"
- [results] Article model input: "All the qPTMplants phosphoprotein sequences were used as an input for the ProtT5‐XL‐U50 pre‐trained pLM"
- [methods] Protein filtering before embedding: "Proteins that are not present in genomic databases or contain missing or non‐canonical amino‐acid sequences were discarded"
- [results] Residue and protein embedding extraction: "we extracted the Ser, Thr, and Tyr residue embedding vectors and the protein‐wise average embedding vector"
