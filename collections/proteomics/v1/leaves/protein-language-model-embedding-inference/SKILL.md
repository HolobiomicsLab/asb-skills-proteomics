---
name: protein-language-model-embedding-inference
description: Use when a phosphorylation-site classifier needs ProtT5-XL-U50 residue
  and protein-level embedding features computed in batches from whole-proteome or
  qPTMplants FASTA files.
when_to_use_negative:
- When you only have already-computed embedding pickle files with a label column —
  proceed directly to PhosBoost labeling/training rather than re-embedding
- When a downstream method requires features other than ProtT5-XL-U50 embeddings (e.g.
  one-hot encodings or DeepPhos CNN features)
- When input proteins contain non-canonical or missing amino acids — these sequences
  are discarded before embedding per the article's filter steps
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: ProtT5-XL-U50
  role: Pretrained protein language model whose per-residue and mean-pooled outputs
    constitute the embedding features
  repo: https://github.com/agemagician/ProtTrans
- name: ProtTrans
  role: Source repository for the ProtT5 encoder and the embed_ProtT5.ipynb notebook
    modified for PhosBoost
  repo: https://github.com/agemagician/ProtTrans
- name: PhosBoost
  role: Downstream consumer of the generated embedding pickle files (modified notebook
    ships with PhosBoost; outputs land in data/embeddings/)
  repo: https://github.com/eporetsky/PhosBoost
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:86c18da35ae4f5538e2027b21923c42fd62e061c611dac691d1be629e2b0f807
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/agemagician/ProtTrans
  tools:
  - CatBoost
  - DeepPhos
  - PhosBoost
  - PhosphoLingo
  - ProtT5-XL-U50
  - ProtTrans
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# protein-language-model-embedding-inference

## Summary

Generate per-residue and protein-wise average embedding vectors with the pretrained ProtT5-XL-U50 protein language model for every Ser/Thr/Tyr residue in a plant proteome FASTA, producing compressed pickle files ready for downstream PhosBoost training and prediction.

## When to use

Use when a phosphorylation-site classifier needs ProtT5-XL-U50 residue and protein-level embedding features computed in batches from whole-proteome or qPTMplants FASTA files.

## When NOT to use

- When you only have already-computed embedding pickle files with a label column — proceed directly to PhosBoost labeling/training rather than re-embedding
- When a downstream method requires features other than ProtT5-XL-U50 embeddings (e.g. one-hot encodings or DeepPhos CNN features)
- When input proteins contain non-canonical or missing amino acids — these sequences are discarded before embedding per the article's filter steps

## Inputs

- Protein FASTA file (e.g. Arabidopsis.qPTMplants.fasta or grass proteome FASTA for Z. mays, T. aestivum, A. sativa, H. vulgare) placed in data/fasta/
- ProtT5-XL-U50 pretrained pLM weights from the ProtTrans repository (huggingface/transformers T5EncoderModel)
- Modified embed_ProtT5.ipynb notebook / embed.batch.sh, embed.merge.sh, embed.sh SLURM scripts

## Outputs

- Compressed python pickle file(s) of per-residue Ser/Thr/Tyr embedding vectors plus protein-wise average embedding vectors, saved to data/embeddings/ (batch intermediates in data/tmp/)
- Optional per-protein average embedding vector set merged across batch jobs

## How to apply

Run a modified version of the ProtTrans embed_ProtT5.ipynb notebook so that the output is a compressed python pickle file of residue and protein-wise average embedding vectors, avoiding manual extraction from h5 files. Embed all Ser, Thr, and Tyr residues of the input FASTA (e.g. Arabidopsis.qPTMplants.fasta) plus one average protein embedding per sequence. For large proteomes, run the embedding in batch mode on CPU nodes (48 threads, 350 GB RAM) as separate SLURM jobs, then merge the batch pickle files from data/tmp/ into data/embeddings/. Set TORCH_HOME/TRANSFORMERS_CACHE to control where the large pretrained pLM weights are stored, and load ProtT5 via the transformers T5Tokenizer/T5EncoderModel with sentencepiece installed.

## Related tools

- **ProtT5-XL-U50** (Pretrained protein language model whose per-residue and mean-pooled outputs constitute the embedding features) — https://github.com/agemagician/ProtTrans
- **ProtTrans** (Source repository for the ProtT5 encoder and the embed_ProtT5.ipynb notebook modified for PhosBoost) — https://github.com/agemagician/ProtTrans
- **PhosBoost** (Downstream consumer of the generated embedding pickle files (modified notebook ships with PhosBoost; outputs land in data/embeddings/)) — https://github.com/eporetsky/PhosBoost

## Examples

```
sbatch embed.batch.sh name.fasta name 30 && sbatch embed.merge.sh name
```

## Evaluation signals

- The output is a single compressed python pickle per input FASTA containing embedding vectors for every Ser, Thr, and Tyr residue plus protein-wise average embeddings, loadable directly by PhosBoost without h5 extraction
- For genome-scale runs, prediction/embedding score files must cover every S/T/Y residue of the input proteome
- Batch mode: the number of data/tmp/ batch pickle files matches the requested split-batch count (e.g. 30), and embed.merge.sh produces one merged file with no missing sequences
- Downstream label.sh succeeds in adding a 'y' column to the merged embedding file, confirming schema compatibility (embedding rows align with qPTMplants site positions)

## Limitations

- Embedding generation was run on CPU nodes (48 threads, 350 GB RAM) rather than GPUs; large RAM is needed so protein length is not limited, which may fail on smaller-memory machines
- The modified notebook produces a PhosBoost-specific pickle schema; raw ProtTrans h5 outputs require separate extraction if reused elsewhere
- Pretrained model weights are large; TORCH_HOME/TRANSFORMERS_CACHE must point to a project models folder to avoid home-directory quota issues

## Evidence

- [readme] PhosBoost was trained using the combined S/T/Y residue embedding vector data and average protein embedding vector data, generated using the pretrained ProtT5-XL-U50 pLM.: "PhosBoost was trained using the combined S/T/Y residue embedding vector data and average protein embedding vector data, generated using the pretrained ProtT5-XL-U50 pLM."
- [readme] Modified ProtTrans notebook produces a pickle ready for training and prediction.: "the generated compressed python pickle file is ready for training and prediction by PhosBoost"
- [readme] CPU batching strategy for large proteomes.: "we used multiple CPU nodes (48 threads, 350GB RAM) to process the input FASTA file in multiple batches"
- [results] Article results confirm which vectors are extracted from the pLM.: "we extracted the Ser, Thr, and Tyr residue embedding vectors and the protein‐wise average embedding vector"
- [methods] Article methods identify the embedding source.: "was obtained from the ProtTrans GitHub repository (https://github.com/agemagician/ProtTrans)"
