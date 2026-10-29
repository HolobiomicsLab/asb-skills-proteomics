---
name: rt-feature-distribution-significance-testing
description: Use when evaluating whether one deep learning RT prediction tool yields
  more accurate delta RT features than another for rescoring PSMs, and whether the
  substitution changes peptide identification counts at a fixed FDR.
when_to_use_negative:
- You only have a single RT predictor's output and no alternative feature to compare
  against — there is nothing to significance-test
- The data are standard tryptic shotgun proteomics rather than immunopeptidomics (unspecific
  cleavage, MHC-bound peptides) — the high-quality peptide definitions and search
  space assumptions differ
- You need to train an RT model from scratch rather than compare already-computed
  feature distributions and their rescoring impact
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: AutoRT
  role: Produces the baseline RTdelta feature via run-specific transfer-learned prediction
    models
  repo: https://github.com/bzhanglab/AutoRT
- name: GPTime
  role: Produces the alternative delta RT feature substituted into rescoring for comparison
- name: Percolator
  role: Rescores PSMs with each delta RT feature configuration (v3.4) to measure the
    performance effect
- name: DeepRescore
  role: Nextflow/Docker pipeline that integrates RTdelta and pDeep2 SA features for
    rescoring
  repo: https://github.com/bzhanglab/DeepRescore
- name: PGA
  role: Implements the target-decoy 1% FDR filtering used to define the comparison
    endpoint (q-value <= 0.01)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201900334
    title: 'DeepRescore: Leveraging Deep Learning to Improve Peptide Identification
      in Immunopeptidomics'
  source_content_hash: sha256:5d1e380e035018e3d1f7252720269a2c0dc3f275f67823e4e4bcb7ec997f300d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bzhanglab/AutoRT
  tools:
  - AutoRT
  - DeepRescore
  - Docker
  - GPTime
  - MS-GF+
  - MaxQuant
  - NextFlow
  - PGA
  - Percolator
  - ProteoWizard
  - X!Tandem
  - pDeep2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# rt-feature-distribution-significance-testing

## Summary

Statistically compares retention-time-derived features (RTdelta = |RTobserved - RTpredicted|) from two RT predictors, e.g. AutoRT versus GPTime, and quantifies how swapping the feature changes Percolator-based PSM rescoring in DeepRescore for immunopeptidomics datasets.

## When to use

Use when evaluating whether one deep learning RT prediction tool yields more accurate delta RT features than another for rescoring PSMs, and whether the substitution changes peptide identification counts at a fixed FDR.

## When NOT to use

- You only have a single RT predictor's output and no alternative feature to compare against — there is nothing to significance-test
- The data are standard tryptic shotgun proteomics rather than immunopeptidomics (unspecific cleavage, MHC-bound peptides) — the high-quality peptide definitions and search space assumptions differ
- You need to train an RT model from scratch rather than compare already-computed feature distributions and their rescoring impact

## Inputs

- MGF-format MS/MS spectra converted from raw files with ProteoWizard (e.g. D1: 938,131 spectra; D2: 279,700 spectra)
- PSM search results from MS-GF+, Comet, X!Tandem, and MaxQuant against the human RefSeq/customized protein database
- Run-specific AutoRT RT predictions for all PSMs (RTdelta = |RTobserved - RTpredicted|)
- GPTime RT predictions for the same PSMs
- Percolator feature tables including pDeep2 spectral angle (SA) features

## Outputs

- RTdelta statistics table per search engine and dataset for both AutoRT- and GPTime-derived features
- Unique peptide identification counts at 1% FDR (PSM and peptide levels) for each feature configuration
- Paired significance test p-values comparing the two delta RT feature distributions (threshold p<0.01)

## How to apply

For each LC-MS/MS run, fine-tune run-specific AutoRT models by transfer learning from pre-trained base models using high-quality peptides (1% FDR at both PSM and peptide levels), then predict RT for all PSMs and compute RTdelta = |RTobserved - RTpredicted|; generate the alternative feature (e.g. GPTime-derived RTdelta) for the same PSMs. Re-run Percolator (v3.4) rescoring separately for each search engine (MS-GF+, Comet, X!Tandem, MaxQuant) on each dataset, substituting the GPTime delta RT for the AutoRT delta RT while keeping pDeep2 spectral angle and other features fixed. Filter both configurations at 1% FDR (q-value <= 0.01) at PSM and peptide levels using the target-decoy strategy implemented in PGA. Compare the RTdelta distributions and the resulting peptide counts across the group comparisons using paired significance testing at the p<0.01 threshold, and tabulate RTdelta statistics and unique peptide counts per search engine and dataset (e.g. D1 Mel15 HLA class I data from PRIDE PXD004894, D2 A*11:01 data from MassIVE MSV000084442).

## Related tools

- **AutoRT** (Produces the baseline RTdelta feature via run-specific transfer-learned prediction models) — https://github.com/bzhanglab/AutoRT
- **GPTime** (Produces the alternative delta RT feature substituted into rescoring for comparison)
- **Percolator** (Rescores PSMs with each delta RT feature configuration (v3.4) to measure the performance effect)
- **DeepRescore** (Nextflow/Docker pipeline that integrates RTdelta and pDeep2 SA features for rescoring) — https://github.com/bzhanglab/DeepRescore
- **PGA** (Implements the target-decoy 1% FDR filtering used to define the comparison endpoint (q-value <= 0.01))

## Examples

```
python -c "from scipy.stats import wilcoxon; print(wilcoxon(autort_rtdelta, gptime_rtdelta))"
```

## Evaluation signals

- GPTime-derived RTdelta values differ significantly from AutoRT-derived RTdelta values in all three group comparisons at p<0.01
- Direction of effect matches the paper: substituting GPTime delta RT causes obvious performance reductions across all four search engines in both D1 and D2, whereas all three comparisons are significant with AutoRT and only one with GPTime
- Per-engine unique peptide counts at 1% FDR (q-value <= 0.01) are reported for both feature configurations and both datasets
- AutoRT training used the correct high-quality peptide definition: 1% FDR at both PSM and peptide levels per at least three of four search engines (or per engine separately in the real-application setting)
- FDR filtering is applied at both PSM and peptide levels via the target-decoy strategy, not at a single level

## Limitations

- GPTime-derived features yielded only one significant group comparison out of three, so non-significant per-engine differences can coexist with overall performance reduction
- Two different high-quality peptide definitions exist for AutoRT training (at least three of four search engines vs. per engine separately), which changes the fine-tuning set and must be stated
- Results depend on run-specific transfer learning per LC-MS/MS run; AutoRT prediction requires GPU resources (nvidia-docker) in related pipelines

## Evidence

- [readme] AutoRT is the deep learning RT prediction tool providing the delta RT feature: [evidence span withheld — anchor status: not_found]
