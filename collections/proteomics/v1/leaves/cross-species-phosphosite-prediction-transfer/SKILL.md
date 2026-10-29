---
name: cross-species-phosphosite-prediction-transfer
description: Use when you need genome-wide phosphosite probability scores for a species
  lacking its own trained model, and have a trained PhosBoost .pkl plus a proteome
  FASTA file.
when_to_use_negative:
- You need high-precision (not high-recall) predictions — PhosBoost has a precision-recall
  tradeoff with more predicted false positives than PhosphoLingo or DeepPhos
- The target residues are Tyr and you rely on inferred-positive enrichment — PhosBoost
  was less informative for Tyr inferred positives, possibly due to higher label imbalance
- Input is already a labeled training dataset for a new species — this skill covers
  prediction/transfer only, not model training or hyperparameter tuning
edam_operation: http://edamontology.org/operation_0417
edam_topics:
- http://edamontology.org/topic_3510
- http://edamontology.org/topic_0601
tools:
- name: PhosBoost
  role: Trained S/T and Y stacking classifiers applied 'all' mode to every target
    residue of the embedded proteomes
  repo: https://github.com/eporetsky/PhosBoost
- name: ProtT5-XL-U50
  role: Generates Ser/Thr/Tyr residue and protein-wise average embedding vectors for
    target-species proteomes
  repo: https://github.com/agemagician/ProtTrans
- name: DIAMOND
  role: Pairwise alignment of 31-residue phosphosite peptides against experimentally
    derived sites to relabel predicted false positives as inferred positives
- name: CatBoost
  role: Gradient-boosting-tree base classifiers inside the PhosBoost stacking model
    used for transfer prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:7507d1497f612e1169d1659aa5c688a03e6b72707dc6b1a84705653ce71685ba
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
  edam_operation: http://edamontology.org/operation_0417
  edam_topics:
  - http://edamontology.org/topic_3510
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-species-phosphosite-prediction-transfer

## Summary

Applies PhosBoost S/T and Y stacking models (CatBoost base classifiers + logistic regression metaclassifier) trained on the complete qPTMplants dataset to proteomes of other plant species (e.g. wheat, oat, barley, maize), enabling genome-wide phosphorylation predictions with ProtT5-XL-U50 embeddings, optionally with DIAMOND inferred-positive annotation, exportable as GFF3 genome-browser tracks.

## When to use

Use when you need genome-wide phosphosite probability scores for a species lacking its own trained model, and have a trained PhosBoost .pkl plus a proteome FASTA file.

## When NOT to use

- You need high-precision (not high-recall) predictions — PhosBoost has a precision-recall tradeoff with more predicted false positives than PhosphoLingo or DeepPhos
- The target residues are Tyr and you rely on inferred-positive enrichment — PhosBoost was less informative for Tyr inferred positives, possibly due to higher label imbalance
- Input is already a labeled training dataset for a new species — this skill covers prediction/transfer only, not model training or hyperparameter tuning

## Inputs

- Target-species proteome FASTA files (e.g. Z. mays, T. aestivum, A. sativa, H. vulgare)
- Trained PhosBoost model pickles from data/models/ (e.g. qPTMplants.ST.joblib.pkl, qPTMplants.Y.joblib.pkl)
- Pretrained ProtT5-XL-U50 pLM weights (via ProtTrans embed_ProtT5 pipeline)
- Optional: qPTMplants phosphosite list for DIAMOND inferred-positive annotation

## Outputs

- Per-site predicted phosphorylation probability scores per species (csv/tsv in data/preds/)
- GFF3 phosphosite prediction track for JBrowse/GrainGenes/MaizeGDB genome browsers
- Compressed pickle files of residue and protein-wise average ProtT5 embeddings

## How to apply

For each target-species proteome FASTA, generate per-residue (Ser/Thr/Tyr) and protein-wise average embedding vectors with the pretrained ProtT5-XL-U50 pLM using the modified ProtTrans embed pipeline, run in batches on multi-thread CPU nodes (e.g. 48 threads, 350GB RAM via sbatch embed.batch.sh) and merged into compressed pickle files. Load the trained PhosBoost S/T stacking model (.pkl from data/models/, two CatBoost base classifiers with balanced-weight logistic regression metaclassifier) and predict on all S/T residues of each embedding file ('all' mode via sbatch predict.sh), then repeat with the Y model on all Tyr residues. Write per-site predicted probabilities to output csv/tsv files in data/preds/. Optionally supplement with a DIAMOND pairwise-alignment step that relabels predicted false positives matching experimentally derived phosphosites (excluding self-matches) as inferred positives. Convert predictions (and DIAMOND alignments) to GFF3 for upload to genome browsers such as GrainGenes (T. aestivum IWGSC v2.1, H. vulgare Morex v3, A. sativa Sang v1) or MaizeGDB.

## Related tools

- **PhosBoost** (Trained S/T and Y stacking classifiers applied 'all' mode to every target residue of the embedded proteomes) — https://github.com/eporetsky/PhosBoost
- **ProtT5-XL-U50** (Generates Ser/Thr/Tyr residue and protein-wise average embedding vectors for target-species proteomes) — https://github.com/agemagician/ProtTrans
- **DIAMOND** (Pairwise alignment of 31-residue phosphosite peptides against experimentally derived sites to relabel predicted false positives as inferred positives)
- **CatBoost** (Gradient-boosting-tree base classifiers inside the PhosBoost stacking model used for transfer prediction)

## Examples

```
sbatch embed.batch.sh name.fasta name 30; sbatch embed.merge.sh name; sbatch predict.sh model_qPTMplants.ST.joblib.pkl name 'all'
```

## Evaluation signals

- Prediction score files are produced for all four species and cover every S/T/Y residue of each input proteome
- Embedding batches (e.g. 30 sbatch jobs) merge into a single compressed pickle without truncated sequences — no non-canonical/missing amino acids
- Output csv/tsv probability scores fall in [0,1] and load into the GFF3 converter without errors
- GFF3 tracks upload successfully to genome browsers (GrainGenes T. aestivum IWGSC v2.1, H. vulgare Morex v3, A. sativa Sang v1; MaizeGDB)
- Recall on benchmark data exceeds alternatives (S/T recall .78 vs .56/.14; Tyr recall .6 vs 0 for PhosphoLingo/DeepPhos) with comparable AUPRC (.54 vs .56/.42)

## Limitations

- Precision-recall tradeoff: increased number of predicted false positives compared to PhosphoLingo and DeepPhos
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, possibly due to higher label imbalance
- Label uncertainty and mislabeling from incomplete high-throughput phosphoproteomic detection is only partially addressed by the DIAMOND annotation step
- Claims that performance improves with input data size and that balanced class weights address imbalance are stated as unvalidated hypotheses

## Evidence

- [abstract] transferability of models: "We provide evidence to show that PhosBoost models are transferable across species and scalable for genome-wide protein phosphorylation predictions"
- [readme] embedding generation: "we used multiple CPU nodes (48 threads, 350GB RAM) to process the input FASTA file in multiple batches"
- [results] DIAMOND relabeling: "Phosphosites matching any experimentally derived phosphosite, excluding self‐matches, were relabeled as inferred positives (IP)"
- [abstract] recall advantage: "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively) while maintaining a competitive area under the precision-recall curve"
- [discussion] precision tradeoff: "PhosBoost presents a precision‐recall tradeoff associated with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos"
