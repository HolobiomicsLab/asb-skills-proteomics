---
name: msms-spectrum-prediction-spectral-angle-calculation
description: Use when rescoring peptide-spectrum matches from immunopeptidomics searches
  (pep.xml + MGF inputs) and you want a peptide-specific, search-engine-independent
  quality feature based on predicted-vs-experimental spectral similarity.
when_to_use_negative:
- You only have raw unsearched spectra and no search engine PSM results to rescore
  — DeepRescore rescoring requires existing identifications plus MGF data
- Your MS/MS data come from an instrument/fragmentation type with no pre-trained pDeep2
  model (e.g. not matching the Lumos pre-trained model), so spectral predictions would
  be unreliable
- You need de novo peptide sequencing rather than rescoring of database-search PSMs
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: pDeep2
  role: Deep learning model that predicts the MS/MS spectrum for each PSM's peptide
    using a pre-trained instrument-specific model (Lumos)
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: DeepRescore
  role: Nextflow/Docker pipeline that computes the SA feature, integrates the three
    feature sets, and runs Percolator rescoring
  repo: https://github.com/bzhanglab/DeepRescore
- name: Percolator
  role: Semi-supervised learner that combines SA with search-engine-specific and independent
    features to produce new PSM scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:e0f5cae85301b1f6644f3c425c3e7c79d04264f8aeb6eb3f632ec78f64b06ef2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AutoRT
  - DeepRescore
  - Docker
  - NextFlow
  - Percolator
  - pDeep2
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msms-spectrum-prediction-spectral-angle-calculation

## Summary

A PSM rescoring feature from the DeepRescore immunopeptidomics workflow: pDeep2 predicts the MS/MS spectrum of each candidate peptide and the normalized spectral angle (SA) between predicted and experimental spectra is computed as a deep learning-derived feature for Percolator rescoring.

## When to use

Use when rescoring peptide-spectrum matches from immunopeptidomics searches (pep.xml + MGF inputs) and you want a peptide-specific, search-engine-independent quality feature based on predicted-vs-experimental spectral similarity.

## When NOT to use

- You only have raw unsearched spectra and no search engine PSM results to rescore — DeepRescore rescoring requires existing identifications plus MGF data
- Your MS/MS data come from an instrument/fragmentation type with no pre-trained pDeep2 model (e.g. not matching the Lumos pre-trained model), so spectral predictions would be unreliable
- You need de novo peptide sequencing rather than rescoring of database-search PSMs

## Inputs

- MS/MS data in MGF format (raw files converted with ProteoWizard)
- Search engine identification results (e.g. Comet .pep.xml)
- PSM peptide sequences and their experimental spectra
- Instrument and collision-energy parameters (e.g. --ms_instrument Lumos, --ms_energy 0.34)

## Outputs

- Per-PSM spectral angle (SA) feature values
- Rescored PSM table (*_psm_final.tsv) with q-value, Percolator_score, Delta_RT, and SA columns at 1% FDR
- Rescored peptide table (*_pep_final.tsv) with q-value, Percolator_score, Delta_RT, and SA columns at 1% FDR

## How to apply

Run the DeepRescore pipeline (Nextflow + Docker with nvidia-docker, GPU required) after database searching, e.g. with Comet (--se comet) supplying the pep.xml and MGF files. For every PSM's peptide, predict the MS/MS spectrum with pDeep2 using the instrument-specific pre-trained model (e.g. the Lumos model, with collision energy such as --ms_energy 0.34). Calculate the normalized spectral angle between predicted and experimental spectra as SA = 1 − 2cos⁻¹(S1·S2)/π, where S1 and S2 are the intensity vectors of the two spectra. Feed the SA feature together with the search-engine-specific features (e.g. Score, PEP, Delta Score for Comet), the search-engine-independent features, and the AutoRT-derived Delta_RT feature into Percolator to rescore all target and decoy PSMs. The rationale is that high spectral similarity indicates peptides whose fragments are well explained by the model, improving discrimination; the paper reports the performance gain was largely driven by these deep learning-derived features.

## Related tools

- **pDeep2** (Deep learning model that predicts the MS/MS spectrum for each PSM's peptide using a pre-trained instrument-specific model (Lumos)) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **DeepRescore** (Nextflow/Docker pipeline that computes the SA feature, integrates the three feature sets, and runs Percolator rescoring) — https://github.com/bzhanglab/DeepRescore
- **Percolator** (Semi-supervised learner that combines SA with search-engine-specific and independent features to produce new PSM scores)

## Examples

```
nextflow run DeepRescore.nf --se comet --ms_instrument Lumos --ms_energy 0.34 --decoy_prefix XXX_
```

## Evaluation signals

- Both *_psm_final.tsv and *_pep_final.tsv exist in out_dir/DeepRescore_results and contain q-value, Percolator_score, Delta_RT, and SA columns
- Reported PSM and peptide identifications pass the 1% FDR threshold (q-value <= 0.01)
- SA values fall in the valid normalized spectral-angle range and are non-degenerate (not constant across PSMs)
- Rescoring with SA increases unique peptide identifications versus search-engine-only rescoring (e.g. Comet identified 23400 unique peptides for D1 before DeepRescore augmentation)

## Limitations

- DeepRescore requires a GPU environment (nvidia-docker) because the deep learning feature computation (pDeep2, AutoRT) runs on GPU
- Only four search engines are supported (MS-GF+, Comet, X!Tandem, MaxQuant), so the feature extraction is tied to their output formats
- The spectral angle quality depends on the instrument-specific pre-trained model; the paper used a pre-trained Lumos model for its datasets (D1: 938,131 spectra; D2: 279,700 spectra)

## Evidence
