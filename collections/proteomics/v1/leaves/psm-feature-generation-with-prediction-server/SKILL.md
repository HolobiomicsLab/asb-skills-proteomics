---
name: psm-feature-generation-with-prediction-server
description: Use when raw-file search engine results (e.g. HLA immunopeptidomics PSMs)
  need ML-based fragment intensity predictions to build features for mokapot/percolator
  rescoring, as in Oktoberfest's Rescoring job type.
when_to_use_negative:
- The job type is 'SpectralLibraryGeneration' or 'CollisionEnergyCalibration' rather
  than 'Rescoring' — different Oktoberfest functions (pp.digest, pp.gen_lib, pp.annotate_spectral_library)
  apply
- Search results already contain ML rescoring features and only the FDR estimation
  step is needed (call re.rescore_with_mokapot directly)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Python pipeline providing re.generate_features for PSM feature generation
    in Rescoring jobs
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Koina
  role: Prediction server (default koina.wilhelmlab.org:443) queried to retrieve fragment
    intensity predictions from Prosit models
- name: Prosit
  role: Deep-learning model for peptide property (spectral/fragment intensity) prediction
    used to create in silico reference features
- name: spectrum_fundamentals
  role: Provides theoretical peptide/ion mass calculation and spectra annotation underlying
    feature generation
  repo: https://github.com/wilhelm-lab/spectrum_fundamentals
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:6d1cc0a7581f563708b3cdc4ab427866b70566dce2f3e8ef12fa8fbfcf9b24f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Prosit
  - Python
  - mokapot
  - spectrum_fundamentals
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# psm-feature-generation-with-prediction-server

## Summary

Generates rescoring features for PSMs by interfacing a prediction server (Koina) running Prosit deep-learning models to predict peptide properties (fragment intensities), which are combined with search engine results to enable data-driven rescoring in Oktoberfest.

## When to use

Use when raw-file search engine results (e.g. HLA immunopeptidomics PSMs) need ML-based fragment intensity predictions to build features for mokapot/percolator rescoring, as in Oktoberfest's Rescoring job type.

## When NOT to use

- The job type is 'SpectralLibraryGeneration' or 'CollisionEnergyCalibration' rather than 'Rescoring' — different Oktoberfest functions (pp.digest, pp.gen_lib, pp.annotate_spectral_library) apply
- Search results already contain ML rescoring features and only the FDR estimation step is needed (call re.rescore_with_mokapot directly)

## Inputs

- Search engine output files (PSM results) from HLA raw-file searches
- Raw mass spectrometry files (Thermo RAW, converted via pp.convert_raw_to_mzml and pp.convert_d_to_hdf)
- Oktoberfest configuration file with type 'Rescoring' and inputs block specifying search results and spectra paths
- Access to a prediction server (e.g. koina.wilhelmlab.org:443) serving Prosit models

## Outputs

- Rescoring feature files per PSM (predicted vs. observed fragment intensities and related ML features)
- Feature files suitable for re.rescore_with_mokapot / re.rescore_with_percolator

## How to apply

Write an Oktoberfest configuration with job type 'Rescoring' and specify the search results and spectra paths (spectra_type 'raw') in the inputs block. Convert and load search engine results with pp.convert_search and pp.load_search, then call re.generate_features to build rescoring features. Query the prediction server (default koina.wilhelmlab.org:443) running Prosit models to retrieve predicted fragment intensities for candidate PSM peptides. During peak annotation, apply the default fragment mass tolerances (20 for FTMS, 40 for TOF, 0.35 for ITMS) between theoretical and experimentally observed fragment masses. The generated features are then consumed downstream by re.rescore_with_mokapot for target-decoy FDR estimation.

## Related tools

- **Oktoberfest** (Python pipeline providing re.generate_features for PSM feature generation in Rescoring jobs) — https://github.com/wilhelm-lab/oktoberfest
- **Koina** (Prediction server (default koina.wilhelmlab.org:443) queried to retrieve fragment intensity predictions from Prosit models)
- **Prosit** (Deep-learning model for peptide property (spectral/fragment intensity) prediction used to create in silico reference features)
- **spectrum_fundamentals** (Provides theoretical peptide/ion mass calculation and spectra annotation underlying feature generation) — https://github.com/wilhelm-lab/spectrum_fundamentals

## Examples

```
import oktoberfest as ok; ok.pp.convert_search(...); ok.pp.load_search(...); ok.re.generate_features(config, predictor=ok.pr.Predictor(server='koina.wilhelmlab.org:443'))
```

## Evaluation signals

- Feature files are produced for all PSMs and are accepted by re.rescore_with_mokapot without errors
- Fragment mass tolerances applied during peak annotation match the configured defaults (20 FTMS / 40 TOF / 0.35 ITMS)
- Prediction server responses are retrieved successfully from koina.wilhelmlab.org:443 for all candidate peptides
- Downstream rescoring with mokapot reproduces/improves published sensitivity gains at fixed FDR (e.g. 1% PSM-level FDR)

## Limitations

- Runtime scales linearly with the number of PSMs; large cohorts (e.g. 15 HLA MS files) take ~1 h single-process, mitigated to ~15 min with multiprocessing
- Requires network access to a Koina prediction server; the default server (koina.wilhelmlab.org:443) is a shared external dependency
- Fragment mass tolerance settings (20 FTMS / 40 TOF / 0.35 ITMS) assume the corresponding instrument types; other instrument configurations may need adjusted tolerances

## Evidence

- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [methods] re.generate_features re.merge_input re.rescore_with_mokapot: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot"
- [methods] Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [readme] It offers an end to end pipeline that takes search results, predicts peptide properties using koina, plots summaries and quality control figures and performs FDR estimation with either mokapot or percolator.: [evidence span withheld — anchor status: external_doc]
- [abstract] ML and deep learning (DL) models for peptide property prediction such as Prosit have enabled the creation of high quality in silico reference libraries: [evidence span withheld — anchor status: not_found]
