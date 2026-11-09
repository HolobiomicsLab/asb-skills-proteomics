---
name: target-decoy-fdr-estimation
description: Use when annotated PSMs have been rescored with Prosit/Koina-derived
  features (e.g. spectral angle, intensity features, merged MaxQuant scores) and confidently
  identified peptides/PSMs must be counted at 1% FDR via target/decoy competition.
when_to_use_negative:
- The input is raw spectra or an unannotated search result — FDR estimation is the
  final target/decoy competition step, not a replacement for peak annotation and feature
  generation
- The goal is spectral library generation or collision energy calibration (Oktoberfest
  job types 'SpectralLibraryGeneration' or 'CollisionEnergCalibration') rather than
  rescoring with FDR estimation
- The dataset lacks a target/decoy structure (no decoy sequences generated with specialAas
  'KR'), so q-values cannot be estimated
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Oktoberfest
  role: Pipeline providing re.rescore_with_mokapot and re.rescore_with_percolator
    for target/decoy separation and FDR estimation on PSM and peptide level
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Default FDR estimation engine used by re.rescore_with_mokapot (fdr_estimation_method
    'mokapot')
- name: Percolator
  role: Alternative external FDR estimator invoked via re.rescore_with_percolator
    (Andromeda+Percolator condition)
- name: Prosit
  role: Provides the fragment intensity/iRT prediction features whose added discriminating
    power FDR estimation converts into additional identifications
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, ssl true) interfaced via pr.Predictor
    to retrieve Prosit predictions upstream of rescoring
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:4574bf423bc32c442d9f207512075339df4ee82c78cb9c251937c46562ad60c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Percolator
  - Prosit
  - Python
  - mokapot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# target-decoy-fdr-estimation

## Summary

Perform target/decoy separation and false discovery rate (FDR) estimation on PSM and peptide level within the Oktoberfest rescoring pipeline, using mokapot (default) or Percolator as the competing estimator, following Prosit-feature-based rescoring of search engine results.

## When to use

Use when annotated PSMs have been rescored with Prosit/Koina-derived features (e.g. spectral angle, intensity features, merged MaxQuant scores) and confidently identified peptides/PSMs must be counted at 1% FDR via target/decoy competition.

## When NOT to use

- The input is raw spectra or an unannotated search result — FDR estimation is the final target/decoy competition step, not a replacement for peak annotation and feature generation
- The goal is spectral library generation or collision energy calibration (Oktoberfest job types 'SpectralLibraryGeneration' or 'CollisionEnergCalibration') rather than rescoring with FDR estimation
- The dataset lacks a target/decoy structure (no decoy sequences generated with specialAas 'KR'), so q-values cannot be estimated

## Inputs

- Rescored PSM feature table from re.generate_features / re.merge_input (Oktoberfest internal format, containing Prosit-derived features plus the native search engine score such as Andromeda)
- Target/decoy search results converted via pp.convert_search / pp.load_search / pp.split_search
- FDR estimation method configuration ('mokapot' default or 'percolator') from the Oktoberfest 'Rescoring' job config

## Outputs

- Target and decoy PSM/peptide scores with q-values at PSM and peptide level (mokapot or percolator output)
- Confident identification counts at 1% FDR per condition/allele/search space (e.g. PSM and peptide tallies)
- Input for gain/loss decomposition via pl.plot_gain_loss and comparison figures (e.g. psm_rescoring_comparison.png, rescoring_runtimes.csv, retention_rate_comparison.csv)

## How to apply

After re.generate_features annotates experimental fragment peaks against Prosit predictions (default mass tolerance 20 FTMS / 40 TOF / 0.35 ITMS, unitMassTolerance 'da' or 'ppm' per mass analyzer, fragmentation_method 'HCD') and re.merge_input combines them with the native search score (e.g. Andromeda), pass the feature table to re.rescore_with_mokapot (fdr_estimation_method default 'mokapot') for target/decoy separation and FDR estimation on PSM and peptide level. As an alternative, re.rescore_with_percolator runs the same estimation externally; the four-allele HLA study used both, with mokapot for cross-checking Percolator results at 1% PSM-level FDR. Apply the estimator consistently across conditions (e.g. MaxQuant-only, Andromeda+Percolator, Andromeda+Prosit+Percolator; or per metaproteomics search space) so that PSM/peptide counts are directly comparable. Special amino acids for decoy generation default to 'KR'. Validate by reproducing published counts, e.g. 30,866 PSMs (MaxQuant), 38,196 (Andromeda+Percolator), 95,451 (Andromeda+Prosit+Percolator), or the C*12:03 gains of ~2574 peptides / 14,318 PSMs.

## Related tools

- **Oktoberfest** (Pipeline providing re.rescore_with_mokapot and re.rescore_with_percolator for target/decoy separation and FDR estimation on PSM and peptide level) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Default FDR estimation engine used by re.rescore_with_mokapot (fdr_estimation_method 'mokapot'))
- **Percolator** (Alternative external FDR estimator invoked via re.rescore_with_percolator (Andromeda+Percolator condition))
- **Prosit** (Provides the fragment intensity/iRT prediction features whose added discriminating power FDR estimation converts into additional identifications)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, ssl true) interfaced via pr.Predictor to retrieve Prosit predictions upstream of rescoring)

## Examples

```
import oktoberfest as ok; ok.re.rescore_with_mokapot(merged_features, fdr_estimation_method='mokapot')
```

## Evaluation signals

- At 1% PSM-level FDR, reproduced counts must match published values: 30,866 (MaxQuant), 38,196 (Andromeda+Percolator), 95,451 (Andromeda+Prosit+Percolator) for the four-allele HLA comparison
- C*12:03 rescoring must yield ~2574 uniquely identified peptides and 14,318 PSMs with only 7 peptides / 49 PSMs lost
- Metaproteomics search-space collapse must be reproduced: MaxQuant 11,724 → 434 PSMs vs rescoring 19,124 → 114,510 PSMs
- Retention-rate check: rescoring retains 15,573/19,124 (81%) human-only PSMs vs 3.7% (434) for MaxQuant, i.e. a 68% reduction in losses
- Estimated FDR must be computed at both PSM and peptide level, as configured by fdr_estimation_method

## Limitations

- FDR estimates depend on the target/decoy strategy; decoy generation uses specialAas defaulting to 'KR', so proteins lacking K/R cleavage sites may yield unreliable decoys
- Only two FDR estimation backends are supported ('percolator' or 'mokapot'); percolator requires the external tool to be installed
- Comparative PSM/peptide counts are only meaningful when the same fdr_estimation_method is used across all compared conditions

## Evidence

- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] rescoring functions: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
- [readme] FDR estimation within the end-to-end pipeline: [evidence span withheld — anchor status: external_doc]
- [methods] decoy generation settings: "specialAas                 | Special amino acids for decoy generation; default = "KR""
