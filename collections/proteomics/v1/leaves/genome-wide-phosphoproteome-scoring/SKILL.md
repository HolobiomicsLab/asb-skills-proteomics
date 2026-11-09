---
name: genome-wide-phosphoproteome-scoring
description: Use when you have a complete proteome FASTA (e.g. wheat, oat, barley,
  maize) and pre-trained PhosBoost models, and need phosphorylation probability scores
  for all S/T/Y residues plus browser visualization tracks.
when_to_use_negative:
- You are training or hyperparameter-tuning a new PhosBoost model on labeled qPTMplants
  data — this skill is prediction-only on a fixed trained model
- You only have a small set of peptide windows already centered on candidate phosphosites
  (length-31 windows) rather than a whole proteome — use per-site scoring instead
- High precision is required with minimal false positives — PhosBoost has a precision-recall
  tradeoff with more predicted false positives than PhosphoLingo and DeepPhos
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: PhosBoost
  role: Stacking classifier (two CatBoost base classifiers + logistic regression metaclassifier)
    whose trained S/T and Y .pkl models are run in 'all' mode over all S/T/Y residue
    embeddings
  repo: https://github.com/eporetsky/PhosBoost
- name: ProtT5-XL-U50
  role: Pre-trained protein language model used (via modified ProtTrans embed_ProtT5.ipynb)
    to generate residue and protein-wise average embedding vectors for the input proteome
  repo: https://github.com/agemagician/ProtTrans
- name: CatBoost
  role: Gradient-boosting tree base classifiers inside the PhosBoost stacking model
    used for per-site probability prediction
- name: scikit-learn
  role: Provides the logistic regression metaclassifier and train_test_split utilities
    underpinning the models being applied
- name: DIAMOND
  role: Pairwise alignment step supplementing PhosBoost predictions with inferred-positive
    phosphosite annotations for the GFF3 track
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:49389f93b64dd774aab08c3ab6a690e960d52130bd4f44c9995b6b6205968388
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - CatBoost
  - DIAMOND
  - PhosBoost
  - ProtT5-XL-U50
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genome-wide-phosphoproteome-scoring

## Summary

Applies trained PhosBoost S/T and Y stacking classifiers (CatBoost base classifiers + logistic regression metaclassifier) to every Ser/Thr/Tyr residue of a whole-proteome FASTA using ProtT5-XL-U50 embeddings, producing genome-wide per-site phosphorylation probability scores exportable as GFF3 genome-browser tracks.

## When to use

Use when you have a complete proteome FASTA (e.g. wheat, oat, barley, maize) and pre-trained PhosBoost models, and need phosphorylation probability scores for all S/T/Y residues plus browser visualization tracks.

## When NOT to use

- You are training or hyperparameter-tuning a new PhosBoost model on labeled qPTMplants data — this skill is prediction-only on a fixed trained model
- You only have a small set of peptide windows already centered on candidate phosphosites (length-31 windows) rather than a whole proteome — use per-site scoring instead
- High precision is required with minimal false positives — PhosBoost has a precision-recall tradeoff with more predicted false positives than PhosphoLingo and DeepPhos

## Inputs

- Proteome FASTA file(s) (e.g. Z. mays, T. aestivum, A. sativa, H. vulgare) placed in data/fasta/
- Trained PhosBoost model pickle files: qPTMplants.ST.joblib.pkl and qPTMplants.Y.joblib.pkl (genome-wide models from data/models/)
- Optional positive phosphosite list (data/sites/, e.g. qPTMplants.sites) for DIAMOND inferred-positive annotation

## Outputs

- Per-site predicted phosphorylation probability score csv/tsv files in data/preds/ covering all S/T and Y residues of each proteome
- GFF3 phosphosite prediction track for JBrowse genome browser (e.g. GrainGenes Triticum aestivum IWGSC v2.1, Hordeum vulgare Morex v3, Avena sativa Sang v1)

## How to apply

Generate Ser/Thr/Tyr residue and protein-wise average embedding vectors for the input FASTA with the pretrained ProtT5-XL-U50 pLM using the modified ProtTrans embed_ProtT5 pipeline, run in batches on multi-thread CPU nodes (e.g. 48 threads, 350 GB RAM, ~30 SLURM batches via embed.batch.sh) and merge the compressed pickle files. Load the trained PhosBoost S/T stacking model (.pkl, two CatBoost base classifiers — one with balanced class weights — plus a balanced-weight logistic regression metaclassifier) from data/models/ and run prediction in 'all' mode on every S/T residue; repeat with the PhosBoost Y model for every Tyr residue. Write per-site predicted probability scores to csv/tsv files in data/preds/. Optionally convert prediction results, with DIAMOND inferred-positive annotations, to a GFF3 track for JBrowse/GrainGenes visualization. Rationale: PhosBoost prioritizes recall (0.78 S/T, 0.6 Y on the A. thaliana qPTMplants test set) and its models are species-transferable, so the same complete-qPTMplants-trained models can score any grass proteome.

## Related tools

- **PhosBoost** (Stacking classifier (two CatBoost base classifiers + logistic regression metaclassifier) whose trained S/T and Y .pkl models are run in 'all' mode over all S/T/Y residue embeddings) — https://github.com/eporetsky/PhosBoost
- **ProtT5-XL-U50** (Pre-trained protein language model used (via modified ProtTrans embed_ProtT5.ipynb) to generate residue and protein-wise average embedding vectors for the input proteome) — https://github.com/agemagician/ProtTrans
- **CatBoost** (Gradient-boosting tree base classifiers inside the PhosBoost stacking model used for per-site probability prediction)
- **scikit-learn** (Provides the logistic regression metaclassifier and train_test_split utilities underpinning the models being applied)
- **DIAMOND** (Pairwise alignment step supplementing PhosBoost predictions with inferred-positive phosphosite annotations for the GFF3 track)

## Examples

```
sbatch embed.batch.sh name.fasta name 30 && sbatch embed.merge.sh name && sbatch predict.sh name all
```

## Evaluation signals

- Prediction score files are produced for all four species (wheat, oat, barley, maize) and cover every S/T/Y residue of each input proteome FASTA (residue count matches the FASTA)
- Scores are per-site probability values from the stacking model (balanced-weight logistic regression metaclassifier over two CatBoost base classifiers), not binary labels
- Converted GFF3 tracks upload successfully to JBrowse genome browsers (GrainGenes Triticum aestivum IWGSC v2.1, Hordeum vulgare Morex v3, Avena sativa Sang v1)
- Per-residue model pairing is correct: the S/T model is run on Ser/Thr residues and the Y model only on Tyr residues
- Embedding generation covers both residue embedding vectors and protein-wise average embedding vectors for each site, as required by PhosBoost input features

## Limitations

- PhosBoost presents a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, possibly due to higher label imbalance in Tyr training data
- Many phosphosites remain undetected by high-throughput phosphoproteomic approaches, limiting negative label reliability for the trained models
- Embedding generation is resource-intensive: large RAM (e.g. 350 GB) CPU nodes were used so long protein sequences are not length-limited

## Evidence

- [readme] PhosBoost was trained using the combined S/T/Y residue embedding vector data and average protein embedding vector data, generated using the pretrained ProtT5-XL-U50 pLM.: "PhosBoost was trained using the combined S/T/Y residue embedding vector data and average protein embedding vector data, generated using the pretrained ProtT5-XL-U50 pLM."
- [readme] The embedding file can be used directly for prediction but lacks a label column; multi-node batched embedding on 48-thread CPUs.: "While it is possible to use GPUs to generate the embedding data, we used multiple CPU nodes (48 threads, 350GB RAM) to process the input FASTA file in multiple batches."
- [abstract] Genome-wide scale and cross-species transferability of the trained models.: "We provide evidence to show that PhosBoost models are transferable across species and scalable for genome-wide protein phosphorylation predictions"
- [results] Residue and protein-wise embedding extraction step for S/T/Y.: "we extracted the Ser, Thr, and Tyr residue embedding vectors and the protein‐wise average embedding vector"
- [discussion] Precision-recall tradeoff with increased false positives.: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
