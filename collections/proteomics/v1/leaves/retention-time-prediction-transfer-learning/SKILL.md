---
name: retention-time-prediction-transfer-learning
description: Use when analyzing LC-MS/MS immunopeptidomics PSMs where a retention-time-derived
  feature (RTdelta) should be added to Percolator rescoring, and run-specific RT prediction
  is needed.
when_to_use_negative:
- When the goal is plain peptide identification without deep learning feature augmentation
  — plain Percolator rescoring (as in MHCquant) already uses search engine features
  alone
- When no GPU is available, since AutoRT-based RT prediction within DeepRescore requires
  nvidia-docker (GPU required)
- When input spectra are not from an LC-MS/MS run with measurable retention times,
  since RTdelta cannot be computed without observed RTs
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoRT
  role: Deep learning RT prediction tool; base models are fine-tuned per LC-MS/MS
    run with transfer learning to predict RTs for all PSMs and derive RTdelta
  repo: https://github.com/bzhanglab/AutoRT
- name: DeepRescore
  role: NextFlow/Docker post-processing pipeline that integrates the RTdelta feature
    with other feature sets and runs Percolator rescoring
  repo: https://github.com/bzhanglab/DeepRescore
- name: NextFlow
  role: Workflow engine used to launch the DeepRescore pipeline (DeepRescore.nf) that
    executes the AutoRT fine-tuning and prediction steps
- name: Docker
  role: Container runtime (with nvidia-docker for GPU) used to deploy AutoRT within
    DeepRescore
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:b86fde02bbf2edacfb1360402df3fa145186d7f61b617b35b80e69521c7f5f2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/AutoRT
  tools:
  - AutoRT
  - DeepRescore
  - Docker
  - NextFlow
  - pDeep2
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# retention-time-prediction-transfer-learning

## Summary

Deep learning-based peptide retention time (RT) prediction with per-run transfer learning, used in DeepRescore to compute the RTdelta feature (|RTobserved − RTpredicted|) that improves Percolator-based rescoring of PSMs in immunopeptidomics data.

## When to use

Use when analyzing LC-MS/MS immunopeptidomics PSMs where a retention-time-derived feature (RTdelta) should be added to Percolator rescoring, and run-specific RT prediction is needed.

## When NOT to use

- When the goal is plain peptide identification without deep learning feature augmentation — plain Percolator rescoring (as in MHCquant) already uses search engine features alone
- When no GPU is available, since AutoRT-based RT prediction within DeepRescore requires nvidia-docker (GPU required)
- When input spectra are not from an LC-MS/MS run with measurable retention times, since RTdelta cannot be computed without observed RTs

## Inputs

- Peptide-spectrum matches (PSMs) from a search engine (e.g., Comet .pep.xml) with observed retention times
- MGF-format MS/MS files (e.g., A1101.mgf) defining the LC-MS/MS run
- High-quality peptides passing 1% FDR (q-value <= 0.01) at PSM and peptide levels, used for per-run fine-tuning
- Pre-trained AutoRT base models trained on 136,791 peptides

## Outputs

- Run-specific fine-tuned AutoRT RT prediction model
- Predicted RT values for all PSMs in the run
- RTdelta (= |RTobserved − RTpredicted|) column integrated into the Percolator-rescored *_psm_final.tsv output

## How to apply

Start from AutoRT deep neural network base models pre-trained on a large public dataset of 136,791 peptides. For each LC-MS/MS run, fine-tune the base model with transfer learning using that run's high-quality peptides — those passing 1% FDR filtering at both PSM and peptide levels (e.g., q-value <= 0.01) — and their observed RTs. Use the run-specific model to predict RTs for all PSMs in the run, then calculate RTdelta = |RTobserved − RTpredicted| for each PSM. Feed the RTdelta feature, together with search engine specific features and search engine independent features, into Percolator to rescore target and decoy PSMs; AutoRT itself supports tryptic, MHC-bound, and PTM peptide prediction.

## Related tools

- **AutoRT** (Deep learning RT prediction tool; base models are fine-tuned per LC-MS/MS run with transfer learning to predict RTs for all PSMs and derive RTdelta) — https://github.com/bzhanglab/AutoRT
- **DeepRescore** (NextFlow/Docker post-processing pipeline that integrates the RTdelta feature with other feature sets and runs Percolator rescoring) — https://github.com/bzhanglab/DeepRescore
- **NextFlow** (Workflow engine used to launch the DeepRescore pipeline (DeepRescore.nf) that executes the AutoRT fine-tuning and prediction steps)
- **Docker** (Container runtime (with nvidia-docker for GPU) used to deploy AutoRT within DeepRescore)

## Examples

```
nextflow run DeepRescore.nf --se comet --ms_instrument Lumos --ms_energy 0.34 --decoy_prefix XXX_ --cpu 4 --memory 8GB
```

## Evaluation signals

- The rescored *_psm_final.tsv output contains the Delta_RT column with computed |RTobserved − RTpredicted| values for each PSM
- Peptides used for transfer learning pass 1% FDR (q-value <= 0.01) at both PSM and peptide levels
- Fine-tuning is performed per LC-MS/MS run (run-specific models), not globally across runs
- Rescored PSMs and peptides filtered at q-value <= 0.01 yield increased identifications relative to search-engine features alone, since deep learning-derived features drove the performance improvement
- Reported PSM/peptide identifications in *_psm_final.tsv and *_pep_final.tsv pass the 1% FDR threshold

## Limitations

- AutoRT fine-tuning within DeepRescore requires GPU support via nvidia-docker; without it the RTdelta feature cannot be computed
- Performance depends on enough high-quality peptides per run passing 1% FDR to support run-specific transfer learning
- The base AutoRT models were trained on 136,791 peptides from public data; transfer learning to a very different LC gradient or instrument may limit prediction accuracy

## Evidence
