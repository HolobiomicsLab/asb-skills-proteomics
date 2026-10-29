---
name: spectral-angle-similarity-scoring
description: Use when rescoring PSMs from immunopeptidomics database searches (Comet,
  MS-GF+, X!Tandem, MaxQuant) and you want to quantify how well a deep learning spectrum
  prediction (pDeep2) matches the observed MS/MS spectrum as a discriminative feature.
when_to_use_negative:
- Input lacks experimental MS/MS spectra (e.g., only peptide sequence lists with no
  MGF/peak data), so no spectral angle can be computed against pDeep2 predictions
- The goal is a plain Percolator rescoring with original search engine features only
  and no GPU, since pDeep2 prediction requires deep learning infrastructure (GPU)
  and dropping SA greatly reduces identifications
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pDeep2
  role: Deep learning model that predicts MS/MS spectra of peptide candidates, which
    are compared to experimental spectra to compute the SA feature
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: DeepRescore
  role: Nextflow/Docker post-processing tool that combines SA and other deep learning-derived
    features with previously used features to rescore PSMs
  repo: https://github.com/bzhanglab/DeepRescore
- name: Percolator
  role: Semi-supervised rescoring engine (v3.4) trained with the SA feature included
    to re-rank PSMs and assign q-values
- name: Comet
  role: Search engine producing the PSM identification files (e.g., A1101.pep.xml)
    whose matches are scored with the SA feature
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:db1b678597b1fd9f7ea2f5202f1353b22dc73c24777132330d408930640809cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AutoRT
  - Comet
  - DeepRescore
  - Docker
  - NextFlow
  - Percolator
  - pDeep2
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spectral-angle-similarity-scoring

## Summary

Computes the spectral angle (SA) feature — the normalized similarity between pDeep2-predicted and experimental MS/MS spectra — and adds it, alongside AutoRT-derived RTdelta, as a deep learning-derived feature for Percolator-based rescoring of PSMs in DeepRescore.

## When to use

Use when rescoring PSMs from immunopeptidomics database searches (Comet, MS-GF+, X!Tandem, MaxQuant) and you want to quantify how well a deep learning spectrum prediction (pDeep2) matches the observed MS/MS spectrum as a discriminative feature.

## When NOT to use

- Input lacks experimental MS/MS spectra (e.g., only peptide sequence lists with no MGF/peak data), so no spectral angle can be computed against pDeep2 predictions
- The goal is a plain Percolator rescoring with original search engine features only and no GPU, since pDeep2 prediction requires deep learning infrastructure (GPU) and dropping SA greatly reduces identifications

## Inputs

- MS/MS spectra in MGF format (e.g., A1101.mgf)
- Search engine identification results (e.g., Comet A1101.pep.xml PSM files)
- pDeep2-predicted MS/MS spectra for candidate peptide sequences
- Percolator-compatible feature table slots for SA and previously used features

## Outputs

- Per-PSM SA (normalized spectral angle) feature values
- Rescored PSM results with q-values (filterable at q-value <= 0.01) at 1% FDR, PSM and peptide levels
- Unique peptide counts (e.g., ablation_summary.csv) comparing full model versus ablated configurations

## How to apply

For each PSM, predict the fragment ion spectrum of the candidate peptide with pDeep2 and compute the normalized spectral angle between the predicted and experimental spectra to yield the SA feature. Compute RTdelta in parallel from AutoRT run-specific transfer-learned models fine-tuned on high-quality peptides passing 1% FDR. Feed SA, RTdelta, plus search-engine specific and search-engine independent features into Percolator (v3.4) within DeepRescore (Nextflow + Docker, GPU required), then filter results by q-value <= 0.01. Ablation analysis shows that keeping only delta RT, SA, and the original search engine score leads to only very minor reductions in identifications, while removing both deep learning features causes greatly reduced peptide identifications — so SA should not be dropped when feature budget is limited.

## Related tools

- **pDeep2** (Deep learning model that predicts MS/MS spectra of peptide candidates, which are compared to experimental spectra to compute the SA feature) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **DeepRescore** (Nextflow/Docker post-processing tool that combines SA and other deep learning-derived features with previously used features to rescore PSMs) — https://github.com/bzhanglab/DeepRescore
- **Percolator** (Semi-supervised rescoring engine (v3.4) trained with the SA feature included to re-rank PSMs and assign q-values)
- **Comet** (Search engine producing the PSM identification files (e.g., A1101.pep.xml) whose matches are scored with the SA feature)

## Evaluation signals

- The full DeepRescore model (with SA and RTdelta) outperforms every ablated configuration at 1% FDR in unique peptide counts
- Removing SA alone causes reduced unique peptide identifications for all search engines in both datasets, confirming SA's dominant contribution
- Rescoring with only delta RT, SA, and the original search engine score gives only very minor reductions versus the full feature set
- Output table rows pass the q-value <= 0.01 filter applied to the q-value column
- Peptide identifications are filtered at 1% FDR at both PSM and peptide levels via the target-decoy strategy

## Limitations

- SA computation requires GPU-accelerated deep learning prediction (nvidia-docker for AutoRT/pDeep2-style models), which is optional/heavy infrastructure in the pipeline
- The performance gain is demonstrated on immunopeptidomics (MHC-binding peptide) datasets D1 (Mel15, 938,131 MS/MS spectra) and D2 (A*11:01 mono-allelic cell line, 279,700 MS/MS spectra); transfer to non-HLA data types is not established here
- High false positive rate risk in immunopeptidomics stems from the inflated no-enzyme search space, so SA rescoring must still be coupled with target-decoy 1% FDR filtering

## Evidence
