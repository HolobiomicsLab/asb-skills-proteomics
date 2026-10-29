---
name: psm-rescoring-with-percolator
description: Use when PSMs from search engines such as Comet, MS-GF+, X!Tandem, or
  MaxQuant need to be rescored to improve peptide identification sensitivity, especially
  for immunopeptidomics (MHC-binding peptide) data.
when_to_use_negative:
- Input is already a Percolator-rescored feature table — rescoring again adds no new
  information
- The search engine output is not one of the supported engines (MS-GF+, Comet, X!Tandem,
  MaxQuant) since DeepRescore cannot extract engine-specific features for other tools
- Only low-confidence PSMs are available with no high-quality peptides (1% FDR) to
  fine-tune the AutoRT run-specific RT models
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRescore
  role: Pipeline that extracts the three feature sets and orchestrates Percolator
    rescoring of target and decoy PSMs
  repo: https://github.com/bzhanglab/DeepRescore
- name: Percolator
  role: Semi-supervised learning tool (v3.4) that integrates all features to produce
    a new combined score for each PSM
- name: AutoRT
  role: Deep learning RT prediction tool; base models fine-tuned per LC-MS/MS run
    by transfer learning to compute the Delta_RT feature
  repo: https://github.com/bzhanglab/AutoRT
- name: pDeep2
  role: Deep learning MS/MS spectrum prediction tool used to compute the normalized
    spectral angle (SA) feature
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: NextFlow
  role: Workflow engine used to run the DeepRescore pipeline
- name: Docker
  role: Containerization platform (with nvidia-docker for GPU) in which DeepRescore
    is deployed
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  - task_005
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:17163b6abb1f8c5f9ed21d7aea54d952f463dbe6e2f368a830c8546bd6ab86f8
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
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# psm-rescoring-with-percolator

## Summary

Rescore peptide-spectrum matches (PSMs) by integrating search-engine-specific features, search-engine-independent features, and deep learning-derived features (AutoRT RTdelta and pDeep2 spectral angle) with Percolator (v3.4), as done in DeepRescore for immunopeptidomics data.

## When to use

Use when PSMs from search engines such as Comet, MS-GF+, X!Tandem, or MaxQuant need to be rescored to improve peptide identification sensitivity, especially for immunopeptidomics (MHC-binding peptide) data.

## When NOT to use

- Input is already a Percolator-rescored feature table — rescoring again adds no new information
- The search engine output is not one of the supported engines (MS-GF+, Comet, X!Tandem, MaxQuant) since DeepRescore cannot extract engine-specific features for other tools
- Only low-confidence PSMs are available with no high-quality peptides (1% FDR) to fine-tune the AutoRT run-specific RT models

## Inputs

- MGF-format MS/MS data converted from raw files with ProteoWizard (e.g., A1101.mgf)
- Search engine identification results (e.g., Comet pep.xml file such as A1101.pep.xml)
- Search engine parameters (e.g., --se comet, --ms_instrument Lumos, --ms_energy 0.34, --decoy_prefix XXX_)
- Pre-trained AutoRT base models (trained on 136,791 peptides) and pDeep2 pre-trained instrument-specific models (e.g., Lumos)

## Outputs

- Rescored PSM table with q-value, Percolator_score, Delta_RT, and SA columns (*_psm_final.tsv in DeepRescore_results folder)
- Rescored peptide table (*_pep_final.tsv in DeepRescore_results folder)
- Unique peptide counts at 1% FDR before and after rescoring

## How to apply

Run the DeepRescore Nextflow/Docker pipeline (nextflow run DeepRescore.nf) with the MGF file and a search engine identification file (e.g., A1101.pep.xml with Comet, --se comet, --decoy_prefix XXX_). Compute the delta RT feature by fine-tuning AutoRT run-specific deep neural network models via transfer learning on high-quality peptides (passing 1% FDR), then calculating RTdelta = |RTobserved − RTpredicted| for each PSM. Compute the spectral angle feature with pDeep2 as SA = 1 − 2cos⁻¹(S1·S2)/π between predicted and experimental spectra. Feed the three feature sets — search-engine specific (e.g., Score, PEP, Delta Score for Comet), search-engine independent, and deep learning-derived (RTdelta, SA) — to Percolator (v3.4) to rescore all target and decoy PSMs. Filter the output by q-value <= 0.01 (1% FDR at both PSM and peptide levels) to obtain the final peptide identifications.

## Related tools

- **DeepRescore** (Pipeline that extracts the three feature sets and orchestrates Percolator rescoring of target and decoy PSMs) — https://github.com/bzhanglab/DeepRescore
- **Percolator** (Semi-supervised learning tool (v3.4) that integrates all features to produce a new combined score for each PSM)
- **AutoRT** (Deep learning RT prediction tool; base models fine-tuned per LC-MS/MS run by transfer learning to compute the Delta_RT feature) — https://github.com/bzhanglab/AutoRT
- **pDeep2** (Deep learning MS/MS spectrum prediction tool used to compute the normalized spectral angle (SA) feature) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **NextFlow** (Workflow engine used to run the DeepRescore pipeline)
- **Docker** (Containerization platform (with nvidia-docker for GPU) in which DeepRescore is deployed)

## Examples

```
nextflow run DeepRescore.nf --se comet --ms_instrument Lumos --ms_energy 0.34 --decoy_prefix XXX_ --cpu 4 --mem 8
```

## Evaluation signals

- Output files *_psm_final.tsv and *_pep_final.tsv exist in out_dir/DeepRescore_results and contain q-value, Percolator_score, Delta_RT, and SA columns
- All reported PSM/peptide identifications pass the 1% FDR threshold (q-value <= 0.01)
- Rescored unique peptide counts exceed pre-rescoring counts (e.g., D1 Comet: 23400 → 35683, a 56% increase)
- Ablation check: removing the deep learning features (Delta_RT, SA) from the Percolator feature set causes greatly reduced peptide identifications for all search engines
- The proportion of peptides commonly identified by all four engines increases after rescoring (D1: 33% → 57%; D2: 34% → 74%)

## Limitations

- GPU (nvidia-docker) is required for the AutoRT and pDeep2 deep learning feature computations
- Only four search engines are supported (MS-GF+, Comet, X!Tandem, MaxQuant); results for other engines cannot be rescored
- The deep learning-derived features dominate the improvement; without them, rescoring gains are greatly reduced, so runs lacking RT or spectrum prediction quality will underperform

## Evidence
