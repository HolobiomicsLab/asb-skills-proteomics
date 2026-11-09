---
name: feature-ablation-sensitivity-analysis
description: Use when a rescoring or classification pipeline adds engineered features
  and you must decide which features drive the performance gain, by retraining the
  model with subsets removed and comparing peptide counts at a fixed FDR.
when_to_use_negative:
- The goal is MHC-peptide binding affinity prediction rather than rescoring feature
  contribution (binding affinity used netMHCpan with percentile rank score <= 2% as
  binder definition)
- The aim is to compare search engines themselves rather than quantify feature contributions
  within a fixed rescoring framework
- No deep learning-derived features (RTdelta, SA) are available, so there is nothing
  to ablate against search-engine features
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepRescore
  role: Nextflow/Docker pipeline that rescores PSMs by combining deep learning-derived
    features with previously used features; provides the example test_data for ablation
    runs
  repo: https://github.com/bzhanglab/DeepRescore
- name: Percolator
  role: Semi-supervised rescoring engine retrained under each feature-subset configuration
    (v3.4) to produce q-values for peptide counting
- name: AutoRT
  role: Deep learning RT prediction tool; run-specific transfer-learned models (fine-tuned
    on peptides passing 1% FDR) predict RT for all PSMs to compute RTdelta
  repo: https://github.com/bzhanglab/AutoRT
- name: pDeep2
  role: MS/MS spectrum prediction tool; predicted spectra are compared to experimental
    spectra to compute the normalized spectral angle (SA) feature
  repo: https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- name: Comet
  role: Database search engine whose PSM output (e.g., A1101.pep.xml) serves as the
    input identification file for the ablation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:ca1fc9794b7b78d6153313513b10fab9948d70a9dbe6f27b30d73272e2239863
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# feature-ablation-sensitivity-analysis

## Summary

Ablation analysis that quantifies how much each feature class (here, deep learning-derived delta RT and spectral angle) contributes to a rescoring model's peptide-identification gains, by retraining Percolator with feature subsets and comparing unique peptide counts at 1% FDR.

## When to use

Use when a rescoring or classification pipeline adds engineered features and you must decide which features drive the performance gain, by retraining the model with subsets removed and comparing peptide counts at a fixed FDR.

## When NOT to use

- The goal is MHC-peptide binding affinity prediction rather than rescoring feature contribution (binding affinity used netMHCpan with percentile rank score <= 2% as binder definition)
- The aim is to compare search engines themselves rather than quantify feature contributions within a fixed rescoring framework
- No deep learning-derived features (RTdelta, SA) are available, so there is nothing to ablate against search-engine features

## Inputs

- Search-engine PSM identification file (e.g., Comet A1101.pep.xml)
- MS/MS spectra in MGF format (e.g., A1101.mgf)
- Deep learning-derived features per PSM: RTdelta (AutoRT predicted vs observed RT) and SA (pDeep2 spectral angle)
- Original search-engine features and previously used search-engine independent features

## Outputs

- Unique peptide counts at q-value <= 0.01 (1% FDR) at PSM and peptide levels for each ablation configuration
- ablation_summary.csv tabulating unique peptide counts for full model, -RTdelta, -SA, DL-features-only (RTdelta + SA + original score), and search-engine-features-only configurations

## How to apply

Run the full DeepRescore pipeline (Nextflow + Docker, GPU for AutoRT) on the search-engine output (e.g., Comet A1101.pep.xml and A1101.mgf) to establish the baseline: compute RTdelta for each PSM as the absolute difference between AutoRT run-specific transfer-learned predictions and observed RT, and SA as the normalized spectral angle between pDeep2-predicted and experimental spectra. Record baseline unique peptide counts after filtering on the q-value column at q-value <= 0.01. Then retrain Percolator (v3.4) four times: with delta RT removed, with SA removed, keeping only delta RT + SA + the original search engine score, and with both deep learning features dropped. Tabulate unique peptide counts for all configurations (ablation_summary.csv) and verify that the full model outperforms every ablated model; in the paper, retaining only RTdelta, SA, and the original PSM score caused only very minor reductions, whereas dropping both deep learning features greatly reduced identifications.

## Related tools

- **DeepRescore** (Nextflow/Docker pipeline that rescores PSMs by combining deep learning-derived features with previously used features; provides the example test_data for ablation runs) — https://github.com/bzhanglab/DeepRescore
- **Percolator** (Semi-supervised rescoring engine retrained under each feature-subset configuration (v3.4) to produce q-values for peptide counting)
- **AutoRT** (Deep learning RT prediction tool; run-specific transfer-learned models (fine-tuned on peptides passing 1% FDR) predict RT for all PSMs to compute RTdelta) — https://github.com/bzhanglab/AutoRT
- **pDeep2** (MS/MS spectrum prediction tool; predicted spectra are compared to experimental spectra to compute the normalized spectral angle (SA) feature) — https://github.com/pFindStudio/pDeep/tree/master/pDeep2
- **Comet** (Database search engine whose PSM output (e.g., A1101.pep.xml) serves as the input identification file for the ablation)

## Examples

```
git clone https://github.com/bzhanglab/DeepRescore && nextflow run DeepRescore/main.nf --mgf A1101.mgf --pep A1101.pep.xml --search_engine comet
```

## Evaluation signals

- The full DeepRescore model yields more unique peptides at 1% FDR than every ablated configuration across all search engines and both datasets
- Removing both deep learning features (RTdelta and SA) causes a greatly reduced unique peptide count relative to the full model
- Keeping only RTdelta, SA, and the original search engine score produces only a very minor reduction, confirming the deep learning features dominate the gain
- All configurations use the same q-value <= 0.01 filter, so counts are comparable
- Results for all five configurations are captured in ablation_summary.csv with consistent schema

## Limitations

- Running AutoRT requires a GPU (nvidia-docker / NVIDIA Container Toolkit) and Nextflow + Docker installation
- Ablation conclusions are tied to immunopeptidomics data (D1: Mel15 HLA class I, 938,131 spectra; D2: A*11:01 mono-allelic cell line, 279,700 spectra) and the four supported search engines (Comet, MS-GF+, X!Tandem, MaxQuant)
- High-quality peptides used for AutoRT transfer learning are defined by 1% FDR filtering, so the feature quality depends on search quality per run

## Evidence

- [methods] The run-specific models were used to predict RTs for all PSMs, and delta RT, denoted as RTdelta, was calculated for each PSM: "The run-specific models were used to predict RTs for all PSMs, and delta RT, denoted as RTdelta, was calculated for each PSM"
- [methods] Users can filter the result based on the column q-value (for example, q-value <= 0.01): "Users can filter the result based on the column `q-value` (for example, q-value <= 0.01)"
- [abstract] the performance improvement was, to a large extent, driven by the deep learning-derived features: "the performance improvement was, to a large extent, driven by the deep learning-derived features"
- [readme] DeepRescore takes as input MS/MS data in MGF format and identification results from a search engine: "**DeepRescore** is an immunopeptidomics data analysis tool that leverages deep learning-derived peptide features to rescore peptide-spectrum matches (PSMs). DeepRescore takes as input MS/MS data in"
