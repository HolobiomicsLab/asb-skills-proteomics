---
name: classifier-probability-score-extraction
description: Use when a trained per-site classifier must produce residue-level predicted
  probability scores (per-Ser/Thr/Tyr phosphosite csv files) as input for recall,
  AUPRC, or F-beta benchmarking against other methods.
when_to_use_negative:
- You only need hard binary phosphosite calls rather than calibrated probability scores
  for threshold-free metric computation.
- The model has not yet been trained — use the model-training skill (tune.sh/train.sh
  with hyperparameter files) before attempting prediction.
- The input is already a per-site probability table; re-running predict.sh would duplicate
  existing predictions.
edam_topics:
- http://edamontology.org/topic_3510
tools:
- name: PhosBoost
  role: Stacking classifier (two CatBoost base classifiers + 5-fold CV logistic regression
    metaclassifier) run via predict.sh to emit per-site probability csv files
  repo: https://github.com/eporetsky/PhosBoost
- name: CatBoost
  role: Gradient-boosting tree base classifiers whose per-site probabilities feed
    the logistic regression metaclassifier
- name: ProtT5-XL-U50
  role: Pre-trained protein language model that generates the residue and protein
    embedding vectors used as prediction input features
  repo: https://github.com/agemagician/ProtTrans
- name: PhosphoLingo
  role: Comparator run (v0.1.0, 'full' setting) on the same test sets to produce probability
    predictions for benchmarking
- name: DeepPhos
  role: Comparator run with default settings on the same test sets for benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pld3.554
    title: 'PhosBoost: Improved phosphorylation prediction recall using gradient boosting
      and protein language models'
  source_content_hash: sha256:adf034e7660396a5322058e98381640d792b8c756eaca0afb757baa56ed7f421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eporetsky/PhosBoost
  tools:
  - CatBoost
  - DeepPhos
  - PhosBoost
  - PhosphoLingo
  - ProtT5-XL-U50
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Classifier probability score extraction

## Summary

Runs trained per-site classifiers (e.g., PhosBoost stacking models on S/T and Y test sets) to emit residue-level predicted probability score files (csv) rather than binary labels, so that recall, AUPRC, and F-beta analyses can be computed downstream.

## When to use

Use when a trained per-site classifier must produce residue-level predicted probability scores (per-Ser/Thr/Tyr phosphosite csv files) as input for recall, AUPRC, or F-beta benchmarking against other methods.

## When NOT to use

- You only need hard binary phosphosite calls rather than calibrated probability scores for threshold-free metric computation.
- The model has not yet been trained — use the model-training skill (tune.sh/train.sh with hyperparameter files) before attempting prediction.
- The input is already a per-site probability table; re-running predict.sh would duplicate existing predictions.

## Inputs

- Stratified A. thaliana qPTMplants test-set embedding pickle files (ProtT5-XL-U50 residue + protein-average embeddings) from data/embeddings/
- Phosphosite label (sites) files for the ST and Y test sets from data/sites/
- Trained PhosBoost model pickles: Arabidopsi.qPTMplants.ST.joblib.pkl and Arabidopsi.qPTMplants.Y.joblib.pkl from data/models/
- Phosphoprotein FASTA file (e.g., Arabidopsis.qPTMplants.fasta) for embedding generation if not already computed

## Outputs

- Per-site predicted probability csv files written to data/preds/ for the ST and Y test sets
- Downstream benchmark_metrics.csv of recall and AUPRC values for PhosBoost, PhosphoLingo, and DeepPhos on S/T and Y

## How to apply

Load the stratified test-set embedding pickle files (ProtT5-XL-U50 residue embeddings plus protein-wise average embedding vectors) and the corresponding phosphosite label files from the PhosBoost repository data folders. Run the trained PhosBoost stacking models — two CatBoost base classifiers (one with balanced class weights, one with equal weights) plus a 5-fold cross-validated logistic regression metaclassifier, saved as .joblib.pkl — via predict.sh on both the ST and Y test sets, passing the embedding file name and the residue flag ('ST' or 'Y'). The prediction step writes per-site predicted probability csv files to data/preds/; retain these files for downstream F-beta analysis instead of thresholding to hard labels. Apply the same probability-extraction procedure to comparator classifiers (PhosphoLingo v0.1.0 under the 'full' setting, DeepPhos with defaults) on the identical test sets so metrics are comparable. Correctness is judged by recomputing recall and AUPRC from the probability files and matching the published benchmarks (S/T recall .78/.56/.14 and AUPRC .54/.56/.42 for PhosBoost/PhosphoLingo/DeepPhos; Tyr recall ~.6 for PhosBoost).

## Related tools

- **PhosBoost** (Stacking classifier (two CatBoost base classifiers + 5-fold CV logistic regression metaclassifier) run via predict.sh to emit per-site probability csv files) — https://github.com/eporetsky/PhosBoost
- **CatBoost** (Gradient-boosting tree base classifiers whose per-site probabilities feed the logistic regression metaclassifier)
- **ProtT5-XL-U50** (Pre-trained protein language model that generates the residue and protein embedding vectors used as prediction input features) — https://github.com/agemagician/ProtTrans
- **PhosphoLingo** (Comparator run (v0.1.0, 'full' setting) on the same test sets to produce probability predictions for benchmarking)
- **DeepPhos** (Comparator run with default settings on the same test sets for benchmarking)

## Examples

```
sbatch predict.sh name ST name
```

## Evaluation signals

- Every emitted prediction row has a predicted probability in [0,1] and corresponds one-to-one with a candidate site in the test-set label file.
- Recomputed S/T recall matches approximately .78/.56/.14 and AUPRC approximately .54/.56/.42 for PhosBoost/PhosphoLingo/DeepPhos on the A. thaliana qPTMplants benchmark.
- PhosBoost Tyr recall is approximately .6, while PhosphoLingo and DeepPhos probability files yield zero Tyr site predictions.
- Prediction csv files for all three classifiers are present in data/preds/ and retained for downstream F-beta analysis.
- All three S/T classifiers should yield a similar AUROC of ~0.88 despite differing AUPRC, confirming probability scores are usable across ranking metrics.

## Limitations

- PhosBoost has a precision-recall tradeoff with an increased number of predicted false positives compared to PhosphoLingo and DeepPhos.
- For Tyr phosphosites, PhosBoost was not as informative at predicting inferred positives, possibly due to higher label imbalance.
- Label uncertainty and mislabeling of phosphosites is only partially addressed (e.g., by the DIAMOND annotation step), limiting reliability of downstream metrics.

## Evidence

- [readme] PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores.: "PhosBoost offers improved performance when recall is prioritized while consistently providing more confident probability scores."
- [abstract] PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively): "PhosBoost achieved higher recall than PhosphoLingo and DeepPhos (.78, .56, and .14, respectively)"
- [results] The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency: "The stacking classifier consists of two CatBoost base classifiers, one classifier using balanced class weights based on label frequency"
- [abstract] PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6: "PhosphoLingo and DeepPhos failed to predict any tyrosine phosphorylation sites, while PhosBoost achieved a recall score of .6"
