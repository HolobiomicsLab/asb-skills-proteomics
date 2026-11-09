---
name: spectral-prediction-feature-generation
description: Use when rescoring PSMs from immunopeptidomics (HLA-bound peptide) database
  searches, where unspecific cleavage inflates the search space and adding RT- and
  spectrum-prediction features can raise sensitivity and confidence of MHC-binding
  peptide and neoantigen identifications.
when_to_use_negative:
- The input is already a rescored/filtered feature table — e.g., PSMs already processed
  by Percolator within MHCquant — rather than raw search-engine output requiring feature
  augmentation.
- The data are from standard tryptic shotgun proteomics rather than immunopeptidomics
  (no enzymatic digestion); the inflated-search-space problem and 8-11 aa MHC-binder
  focus this skill targets do not apply.
- Only a single search engine's identifications are available and no MGF conversion
  of the raw data is possible, since DeepRescore's workflow and AutoRT training assume
  multi-engine results and MGF input.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: DeepRescore
  role: Post-processing tool that combines deep learning-predicted RT and MS/MS spectral
    features with previously used features to rescore PSMs, packaged as Docker + NextFlow
  repo: https://github.com/bzhanglab/DeepRescore
- name: AutoRT
  role: Deep learning retention time prediction; base models are fine-tuned per LC-MS/MS
    run with transfer learning to compute RTdelta for every PSM
  repo: https://github.com/bzhanglab/AutoRT
- name: pDeep2
  role: Deep learning MS/MS spectrum prediction; its spectral angle (SA) between predicted
    and observed spectra serves as a rescoring feature
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: Percolator
  role: Semi-supervised learner that integrates search-engine, search-engine-independent,
    and deep learning-derived features to rescore PSMs
- name: ProteoWizard
  role: Converts raw MS/MS data files to the MGF format DeepRescore consumes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:a62966d8413804e56cc9ee3efa01d437694a7115265014c4d9165041cbc31424
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/DeepRescore
  tools:
  - ANNOVAR
  - AutoRT
  - Customprodbj
  - DeepRescore
  - Docker
  - HLAthena
  - MHCflurry
  - MS-GF+
  - MaxQuant
  - NeoFlow
  - NetMHCpan
  - NextFlow
  - PGA
  - Percolator
  - ProteoWizard
  - X!Tandem
  - pDeep2
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# spectral-prediction-feature-generation

## Summary

Generate deep learning-derived peptide features — run-specific retention time predictions (RTdelta via AutoRT) and MS/MS spectrum prediction spectral angles (SA via pDeep2) — and combine them with search-engine features in Percolator to rescore PSMs in immunopeptidomics data, as implemented by DeepRescore.

## When to use

Use when rescoring PSMs from immunopeptidomics (HLA-bound peptide) database searches, where unspecific cleavage inflates the search space and adding RT- and spectrum-prediction features can raise sensitivity and confidence of MHC-binding peptide and neoantigen identifications.

## When NOT to use

- The input is already a rescored/filtered feature table — e.g., PSMs already processed by Percolator within MHCquant — rather than raw search-engine output requiring feature augmentation.
- The data are from standard tryptic shotgun proteomics rather than immunopeptidomics (no enzymatic digestion); the inflated-search-space problem and 8-11 aa MHC-binder focus this skill targets do not apply.
- Only a single search engine's identifications are available and no MGF conversion of the raw data is possible, since DeepRescore's workflow and AutoRT training assume multi-engine results and MGF input.

## Inputs

- Raw MS/MS data files (e.g., 938,131 MS/MS spectra from PRIDE PXD004894, Mel15 HLA class I dataset D1)
- MGF-format converted spectra files
- PSM/peptide identification results from MS-GF+, Comet, X!Tandem, and/or MaxQuant
- Customized protein FASTA database (human RefSeq 56,659 sequences + variant proteins + 245 contaminants) with decoys
- High-quality peptide list passing 1% FDR for AutoRT run-specific model training

## Outputs

- Rescored PSM table with deep-learning features (RTdelta, spectral angle SA) and q-values, filtered at q-value <= 0.01 (*_psm_final.tsv)
- Rescored peptide table at 1% FDR (*_pep_final.tsv)
- Run-specific AutoRT retention time prediction models
- PDV-importable visualization of spectrum predictions

## How to apply

Convert raw MS/MS files to MGF with ProteoWizard (v3.0.19014), search them with MS-GF+, Comet, X!Tandem, and MaxQuant using unspecific cleavage, 10 ppm precursor tolerance, 0.02 Da MS/MS tolerance, oxidation of methionine (variable), carbamidomethyl of cysteine (fixed), and peptide length 8-25. Define high-quality peptides as those passing 1% FDR filtering at both PSM and peptide levels according to at least three of the four search engines, then fine-tune the AutoRT base deep neural network models (pretrained on 136,791 peptides) per LC-MS/MS run by transfer learning on those peptides and their measured RTs. Use the run-specific models to predict RTs for all PSMs and compute RTdelta per PSM; predict spectra with pDeep2 and compute the spectral angle (SA) between predicted and observed spectra. Run DeepRescore (Docker + NextFlow, GPU required) to integrate search-engine features, search-engine-independent features, and the deep learning-derived RTdelta/SA features in Percolator (v3.4), then filter outputs at q-value <= 0.01 (1% FDR controlled at PSM and peptide levels via the target-decoy strategy in PGA).

## Related tools

- **DeepRescore** (Post-processing tool that combines deep learning-predicted RT and MS/MS spectral features with previously used features to rescore PSMs, packaged as Docker + NextFlow) — https://github.com/bzhanglab/DeepRescore
- **AutoRT** (Deep learning retention time prediction; base models are fine-tuned per LC-MS/MS run with transfer learning to compute RTdelta for every PSM) — https://github.com/bzhanglab/AutoRT
- **pDeep2** (Deep learning MS/MS spectrum prediction; its spectral angle (SA) between predicted and observed spectra serves as a rescoring feature) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **Percolator** (Semi-supervised learner that integrates search-engine, search-engine-independent, and deep learning-derived features to rescore PSMs)
- **ProteoWizard** (Converts raw MS/MS data files to the MGF format DeepRescore consumes)

## Examples

```
git clone https://github.com/bzhanglab/DeepRescore
```

## Evaluation signals

- Rescored outputs are filtered at q-value <= 0.01, with 1% FDR controlled at both PSM and peptide levels via the target-decoy strategy implemented in PGA
- AutoRT run-specific models are trained only on high-quality peptides passing 1% FDR filtering at both PSM and peptide levels in at least three of the four search engines
- Every PSM in the final table carries the two deep-learning features: RTdelta (from AutoRT) and spectral angle SA (from pDeep2)
- Rescoring sensitivity is assessed against search-engine-only baselines (e.g., for D1, Comet/MS-GF+/X!Tandem/MaxQuant identified 23400/22789/20727/17122 unique peptides; the four together 31447 with only 33% common to all)
- Downstream MHC-binder evaluation uses peptides of length 8-11 with NetMHCpan/MHCflurry/HLAthena percentile rank score of 2% or less

## Limitations

- A GPU is required for the AutoRT/pDeep2 deep learning steps (Docker + NextFlow workflow, nvidia-docker >= 2.2.2).
- DeepRescore supports only four search engines (MS-GF+, Comet, X!Tandem, MaxQuant) and takes MS/MS input only in MGF format.
- Immunopeptidomics searches use unspecific cleavage, which inflates the search space and can lead to a high false positive rate and low sensitivity, making stringent FDR control essential.

## Evidence
