---
name: retention-time-prediction-feature-generation
description: 'Use when rescoring immunopeptidomics PSMs with Percolator and a sequence-based
  retention-time feature is needed: fine-tune AutoRT base models per LC-MS/MS run,
  predict RT for each PSM, and compute RTdelta = |RTobserved - RTpredicted|.'
when_to_use_negative:
- The task is database searching or de novo sequencing itself - this skill only post-processes
  existing PSMs and does not replace MS-GF+/Comet/X!Tandem/MaxQuant searches
- No GPU is available for the Docker/NextFlow stack - AutoRT (and pDeep2) inference
  and transfer learning within DeepRescore require a GPU
- A run lacks enough high-quality peptides (1% FDR at PSM and peptide levels) to fit
  a run-specific RT model, or observed RTs are absent from the identification files
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: AutoRT
  role: Sequence-based deep learning RT predictor whose pre-trained base models (136,791
    peptides) are fine-tuned per LC-MS/MS run by transfer learning to produce the
    RTdelta feature for each PSM
  repo: https://github.com/bzhanglab/AutoRT
- name: DeepRescore
  role: NextFlow/Docker pipeline that orchestrates per-run AutoRT fine-tuning, RTdelta
    computation, and integration of all features with Percolator for MS-GF+/Comet/X!Tandem/MaxQuant
    PSMs
  repo: https://github.com/bzhanglab/DeepRescore
- name: Percolator
  role: Semi-supervised rescoring engine (v3.4) that consumes RTdelta alongside search-engine
    and SA features; results are filtered at q-value <= 0.01
- name: pDeep2
  role: Companion spectral-prediction model supplying the SA feature that pairs with
    RTdelta; the two deep learning-derived features jointly dominate the rescoring
    gains
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: GPTime
  role: Alternative RT predictor used to benchmark the AutoRT delta RT feature by
    direct substitution in the Percolator rescoring
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:8d22906c6c680a2ea7d22cab11e5735eb5c521f5af13c1408c01c626f2ecf399
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/AutoRT
  tools:
  - ANNOVAR
  - AutoRT
  - Customprodbj
  - DeepRescore
  - Docker
  - GPTime
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
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# retention-time-prediction-feature-generation

## Summary

Generates the AutoRT-derived RTdelta retention-time feature (|RTobserved - RTpredicted|) from run-specific transfer-learned models, one of the deep learning-derived features that drives DeepRescore's Percolator-based PSM rescoring gains on immunopeptidomics data (datasets D1 and D2).

## When to use

Use when rescoring immunopeptidomics PSMs with Percolator and a sequence-based retention-time feature is needed: fine-tune AutoRT base models per LC-MS/MS run, predict RT for each PSM, and compute RTdelta = |RTobserved - RTpredicted|.

## When NOT to use

- The task is database searching or de novo sequencing itself - this skill only post-processes existing PSMs and does not replace MS-GF+/Comet/X!Tandem/MaxQuant searches
- No GPU is available for the Docker/NextFlow stack - AutoRT (and pDeep2) inference and transfer learning within DeepRescore require a GPU
- A run lacks enough high-quality peptides (1% FDR at PSM and peptide levels) to fit a run-specific RT model, or observed RTs are absent from the identification files

## Inputs

- MGF-format MS/MS files converted from raw data with ProteoWizard v3.0.19014 (e.g., D1: 938,131 Mel15 HLA class I spectra from PRIDE PXD004894; D2: 279,700 A*11:01 spectra from MassIVE MSV000084442)
- Per-PSM search engine identification results (peptide sequence, observed RT, original score) from MS-GF+, Comet, X!Tandem, or MaxQuant
- High-quality peptide lists per run: 1% FDR (q-value <= 0.01) at both PSM and peptide levels according to at least three of four search engines (or per engine)
- Pre-trained AutoRT base models trained on a public dataset of 136,791 peptides

## Outputs

- Run-specific fine-tuned AutoRT retention-time prediction models (one per LC-MS/MS run)
- RTdelta values (|RTobserved - RTpredicted|) for every PSM, appended to the Percolator feature table
- Rescored result tables (*_psm_final.tsv and *_pep_final.tsv) filtered at q-value <= 0.01

## How to apply

Start from MGF files converted with ProteoWizard (v3.0.19014) and per-PSM identifications (sequence, observed RT, score) from MS-GF+, Comet, X!Tandem, or MaxQuant searched with unspecific cleavage, oxidized methionine as variable and carbamidomethyl cysteine as fixed modification, 10 ppm precursor and 0.02 Da MS/MS tolerance, peptide length 8-25. Select high-quality training peptides per run: those passing 1% FDR (q-value <= 0.01) at both PSM and peptide levels, using at least three of the four search engines (Section 3.2) or each engine separately for usability in real applications (Section 3.3). Fine-tune the pre-trained AutoRT base models (trained on 136,791 public peptides) to each LC-MS/MS run via transfer learning, predict RTs for all PSMs, and compute RTdelta = |RTobserved - RTpredicted| for each PSM. Feed RTdelta into Percolator (v3.4) together with search-engine-specific, search-engine-independent, and pDeep2 spectral-angle (SA) features, then filter the output at q-value <= 0.01. Rationale: ablation showed removing delta RT and SA greatly reduced peptide identifications for all engines on D1 and D2, and substituting GPTime-derived delta RT reduced performance across all four engines, confirming AutoRT's run-specific predictions as the more accurate RT feature.

## Related tools

- **AutoRT** (Sequence-based deep learning RT predictor whose pre-trained base models (136,791 peptides) are fine-tuned per LC-MS/MS run by transfer learning to produce the RTdelta feature for each PSM) — https://github.com/bzhanglab/AutoRT
- **DeepRescore** (NextFlow/Docker pipeline that orchestrates per-run AutoRT fine-tuning, RTdelta computation, and integration of all features with Percolator for MS-GF+/Comet/X!Tandem/MaxQuant PSMs) — https://github.com/bzhanglab/DeepRescore
- **Percolator** (Semi-supervised rescoring engine (v3.4) that consumes RTdelta alongside search-engine and SA features; results are filtered at q-value <= 0.01)
- **pDeep2** (Companion spectral-prediction model supplying the SA feature that pairs with RTdelta; the two deep learning-derived features jointly dominate the rescoring gains) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **GPTime** (Alternative RT predictor used to benchmark the AutoRT delta RT feature by direct substitution in the Percolator rescoring)

## Evaluation signals

- Ablation invariant: removing RTdelta and SA from the Percolator feature set greatly reduces unique peptide counts at q-value <= 0.01 for every engine on D1 and D2, while keeping only RTdelta, SA, and the original PSM score gives near-full performance
- Fine-tuned run-specific AutoRT models should yield small RTdelta for the high-quality training peptides; high RTdelta on reported identifications (as for the two original neoantigens missed on D1) suggests false positives
- Substituting GPTime-derived delta RT for AutoRT delta RT must reduce identifications across all four search engines, with RTdelta differences significant (p<0.01) in all three group comparisons
- Output tables (*_psm_final.tsv, *_pep_final.tsv) exist and all reported identifications carry q-value <= 0.01 with FDR controlled at both PSM and peptide levels via the target-decoy strategy in PGA
- Baseline sanity on D1: engine-only identifications are 23400/22789/20727/17122 unique peptides (Comet/MS-GF+/X!Tandem/MaxQuant); DeepRescore-rescored counts should exceed these per engine

## Limitations

- GPU required: DeepRescore runs AutoRT/pDeep2 through Docker + NextFlow with GPU support (nvidia-docker)
- Run-specific fine-tuning depends on the availability of high-quality peptides per run; the paper uses two definitions (at least three of four engines in Section 3.2 versus each engine separately in Section 3.3), which affects applicability in real single-engine workflows
- RTdelta alone is not sufficient - the performance gain depends on pairing with the pDeep2 SA feature; dropping either deep learning-derived feature degrades results
- AutoRT supports tryptic peptides, MHC-bound peptides, and PTM peptides, so the model type must match the experiment; the described workflow targets non-enzymatic immunopeptidomics searches (unspecific cleavage, peptide length 8-25)

## Evidence
