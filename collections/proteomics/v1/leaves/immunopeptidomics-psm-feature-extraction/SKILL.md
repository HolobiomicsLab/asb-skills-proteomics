---
name: immunopeptidomics-psm-feature-extraction
description: Use when raw immunopeptidomics MGF files and search engine outputs (e.g.,
  Comet .pep.xml against A1101-style data) must be rescored with deep learning features
  to increase MHC-binding peptide identification sensitivity.
when_to_use_negative:
- Input identifications already carry a combined Percolator score with q-value <=
  0.01 — the rescoring step and its GPU cost add nothing
- Data are tryptic global proteome or TMT experiments rather than non-enzymatic immunopeptidomics
  (MHC-bound peptide) searches, where the inflated search space motivates rescoring
- No GPU / nvidia-docker environment is available, since AutoRT fine-tuning and pDeep2
  prediction require it
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: DeepRescore
  role: Nextflow/Docker pipeline that extracts the three feature sets and runs Percolator
    rescoring of PSMs
  repo: https://github.com/bzhanglab/DeepRescore
- name: AutoRT
  role: Deep neural network RT prediction tool; base models are fine-tuned per LC-MS/MS
    run with transfer learning to compute the Delta_RT feature
  repo: https://github.com/bzhanglab/AutoRT
- name: pDeep2
  role: Predicts MS/MS spectra of each PSM's peptide to compute the normalized spectral
    angle (SA) feature
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: Percolator
  role: Semi-supervised learning engine that integrates search engine specific, search
    engine independent, and deep learning features into a new combined PSM score
- name: NextFlow
  role: Workflow engine used to orchestrate the DeepRescore pipeline (nextflow run
    DeepRescore.nf)
  repo: https://github.com/bzhanglab/DeepRescore
- name: Comet
  role: Supported search engine whose .pep.xml output and features (Score, PEP, Delta
    Score) serve as input for rescoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:a84db7bd6d49fae190fe30dd576587cc778b9bbb6772897dc9d3cb758f5d99ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/DeepRescore
  tools:
  - AutoRT
  - Comet
  - DeepRescore
  - Docker
  - NextFlow
  - Percolator
  - pDeep2
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# immunopeptidomics-psm-feature-extraction

## Summary

DeepRescore-style feature extraction and Percolator rescoring for peptide-spectrum matches (PSMs) in immunopeptidomics: it integrates search engine specific features, search engine independent features, and deep learning-derived features (AutoRT RTdelta and pDeep2 spectral angle) to produce new PSM scores for 1% FDR filtering.

## When to use

Use when raw immunopeptidomics MGF files and search engine outputs (e.g., Comet .pep.xml against A1101-style data) must be rescored with deep learning features to increase MHC-binding peptide identification sensitivity.

## When NOT to use

- Input identifications already carry a combined Percolator score with q-value <= 0.01 — the rescoring step and its GPU cost add nothing
- Data are tryptic global proteome or TMT experiments rather than non-enzymatic immunopeptidomics (MHC-bound peptide) searches, where the inflated search space motivates rescoring
- No GPU / nvidia-docker environment is available, since AutoRT fine-tuning and pDeep2 prediction require it

## Inputs

- MGF-format MS/MS data file (e.g., A1101.mgf)
- Search engine identification results in pep.xml format (e.g., A1101.pep.xml from Comet)
- Decoy prefix specification (e.g., XXX_)
- Deep learning pre-trained models: AutoRT base models (136,791 peptides) and pDeep2 Lumos model

## Outputs

- *_psm_final.tsv with q-value, Percolator_score, Delta_RT, and SA columns (PSMs passing 1% FDR)
- *_pep_final.tsv with q-value, Percolator_score, Delta_RT, and SA columns (peptides passing 1% FDR)
- Written to the DeepRescore_results folder under out_dir

## How to apply

Run the DeepRescore Nextflow pipeline (nextflow run DeepRescore.nf) inside Docker/nvidia-docker (GPU required) with search engine (e.g., --se comet), instrument (e.g., --ms_instrument Lumos), and collision energy (e.g., --ms_energy 0.34) parameters. Compute the delta RT feature by fine-tuning the AutoRT deep neural network base models (pre-trained on 136,791 peptides) per LC-MS/MS run with transfer learning on peptides passing 1% FDR, then predicting RTs for all PSMs and calculating RTdelta = |RTobserved − RTpredicted|. Compute the spectral angle feature by predicting MS/MS spectra with pDeep2 (pre-trained Lumos model) and calculating SA = 1 − 2cos⁻¹(S1·S2)/π between predicted and experimental spectra. Extract search-engine-specific features (e.g., Score, PEP, Delta Score for Comet) and search-engine-independent features, then feed all three feature sets to Percolator to rescore target and decoy PSMs. Finally, filter rescored PSMs and peptides at 1% FDR (q-value <= 0.01) at both PSM and peptide levels.

## Related tools

- **DeepRescore** (Nextflow/Docker pipeline that extracts the three feature sets and runs Percolator rescoring of PSMs) — https://github.com/bzhanglab/DeepRescore
- **AutoRT** (Deep neural network RT prediction tool; base models are fine-tuned per LC-MS/MS run with transfer learning to compute the Delta_RT feature) — https://github.com/bzhanglab/AutoRT
- **pDeep2** (Predicts MS/MS spectra of each PSM's peptide to compute the normalized spectral angle (SA) feature) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **Percolator** (Semi-supervised learning engine that integrates search engine specific, search engine independent, and deep learning features into a new combined PSM score)
- **NextFlow** (Workflow engine used to orchestrate the DeepRescore pipeline (nextflow run DeepRescore.nf)) — https://github.com/bzhanglab/DeepRescore
- **Comet** (Supported search engine whose .pep.xml output and features (Score, PEP, Delta Score) serve as input for rescoring)

## Examples

```
nextflow run DeepRescore.nf --se comet --ms_instrument Lumos --ms_energy 0.34 --decoy_prefix XXX_ --cpu 4 --memory 8GB
```

## Evaluation signals

- Both *_psm_final.tsv and *_pep_final.tsv exist in out_dir/DeepRescore_results after the run
- Output files contain q-value, Percolator_score, Delta_RT, and SA columns
- Reported PSM and peptide identifications pass the 1% FDR threshold (q-value <= 0.01)
- Delta_RT values are absolute differences |RTobserved − RTpredicted| and SA values fall in [0, 1] as normalized spectral angles
- Rescored identifications should increase unique peptide counts relative to the unrescored search engine output (e.g., Comet's 23,400 unique peptides on D1)

## Limitations

- GPU with nvidia-docker is required for the AutoRT and pDeep2 deep learning feature computation
- Only MS-GF+, Comet, X!Tandem, and MaxQuant search engine outputs are supported as input
- Performance improvement is driven largely by the deep learning-derived features, so runs without usable RT or spectral prediction may see reduced gains

## Evidence

- [abstract] DeepRescore combines peptide features derived from deep learning predictions, namely accurate retention time and MS/MS spectra predictions, with previously used features to rescore peptide-spectrum matches: [evidence span withheld — anchor status: not_found]
