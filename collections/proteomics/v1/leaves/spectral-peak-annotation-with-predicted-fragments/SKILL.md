---
name: spectral-peak-annotation-with-predicted-fragments
description: Use when annotating experimental peaks of search-engine PSMs (e.g. MaxQuant/Andromeda
  results converted via pp.convert_search/pp.load_search) with Prosit fragment intensity
  and iRT predictions from Koina to generate rescoring features.
when_to_use_negative:
- Input is already a completed feature/rescoring table and you only need FDR estimation
  — go directly to re.rescore_with_mokapot
- The goal is spectral library generation (pp.digest, pp.gen_lib) or collision energy
  calibration rather than rescoring search results
- Data were acquired with fragmentation or instrument settings incompatible with the
  configured 'HCD' fragmentation_method and FTMS/TOF/ITMS tolerance defaults
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Python pipeline providing re.generate_features for peak annotation and rescoring
    feature generation
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model producing fragment intensity and iRT predictions used
    to annotate experimental peaks
- name: Koina
  role: Prediction server interfaced by pr.Predictor/pr.Koina to retrieve Prosit model
    predictions
- name: mokapot
  role: Rescoring engine (re.rescore_with_mokapot) performing target/decoy separation
    and FDR estimation on annotated features
- name: spectrum_fundamentals
  role: Support package providing theoretical peptide/ion mass calculation and spectra
    annotation within the Oktoberfest pipeline
  repo: https://github.com/wilhelm-lab/spectrum_fundamentals
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:4bb3e6c66bbaf3d58bd7573e54fb20cacdc2c2d027f1275d6e9df3f4b5a4d4ef
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# Spectral peak annotation with predicted fragments

## Summary

Annotate experimental MS/MS peaks against Prosit-predicted fragment intensities (retrieved via the Koina interface) to build data-driven rescoring features, using mass-analyzer-specific tolerances (20 FTMS / 40 TOF / 0.35 ITMS) with 'da' or 'ppm' unitMassTolerance and fragmentation_method 'HCD'.

## When to use

Use when annotating experimental peaks of search-engine PSMs (e.g. MaxQuant/Andromeda results converted via pp.convert_search/pp.load_search) with Prosit fragment intensity and iRT predictions from Koina to generate rescoring features.

## When NOT to use

- Input is already a completed feature/rescoring table and you only need FDR estimation — go directly to re.rescore_with_mokapot
- The goal is spectral library generation (pp.digest, pp.gen_lib) or collision energy calibration rather than rescoring search results
- Data were acquired with fragmentation or instrument settings incompatible with the configured 'HCD' fragmentation_method and FTMS/TOF/ITMS tolerance defaults

## Inputs

- Unfiltered search engine results (e.g. MaxQuant/Andromeda msms.txt) converted to Oktoberfest internal format via pp.convert_search/pp.load_search
- RAW spectra converted to mzML (ThermoRawFileParser, pp.convert_raw_to_mzml with spectra_type 'raw') and loaded with pp.load_spectra
- Prosit fragment intensity and iRT predictions retrieved via the Koina interface (pr.Predictor/pr.Koina)
- Configuration: fragment mass tolerance per mass analyzer (20 FTMS / 40 TOF / 0.35 ITMS), unitMassTolerance ('da' or 'ppm'), fragmentation_method 'HCD'

## Outputs

- Feature table of annotated PSMs with Prosit-based rescoring features (e.g. intensity/RT similarity features)
- Optionally merged input feature columns (re.merge_input)
- Mokapot-rescored PSM and peptide identifications with target/decoy FDR estimates

## How to apply

Convert search results into Oktoberfest internal format with pp.convert_search/pp.load_search and convert RAW spectra to mzML with ThermoRawFileParser via pp.convert_raw_to_mzml (spectra_type 'raw'), then load with pp.load_spectra. Retrieve Prosit fragment intensity and iRT predictions through the Koina interface (pr.Predictor/pr.Koina) and call re.generate_features to build the rescoring feature table, annotating experimental peaks using the default fragment mass tolerance of 20 (FTMS) / 40 (TOF) / 0.35 (ITMS) with unitMassTolerance 'da' or 'ppm' chosen per mass analyzer and fragmentation_method 'HCD'. Optionally merge additional search-result columns as extra rescoring features via re.merge_input (add_feature_cols). The rationale is that high-quality in silico reference libraries from deep learning models like Prosit provide fragment-intensity similarity signals beyond the native search engine score. Downstream, rescore the annotated PSMs with re.rescore_with_mokapot (fdr_estimation_method default 'mokapot') for target/decoy separation and FDR estimation on PSM and peptide level.

## Related tools

- **Oktoberfest** (Python pipeline providing re.generate_features for peak annotation and rescoring feature generation) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model producing fragment intensity and iRT predictions used to annotate experimental peaks)
- **Koina** (Prediction server interfaced by pr.Predictor/pr.Koina to retrieve Prosit model predictions)
- **mokapot** (Rescoring engine (re.rescore_with_mokapot) performing target/decoy separation and FDR estimation on annotated features)
- **spectrum_fundamentals** (Support package providing theoretical peptide/ion mass calculation and spectra annotation within the Oktoberfest pipeline) — https://github.com/wilhelm-lab/spectrum_fundamentals

## Examples

```
import oktoberfest as ok; from oktoberfest import rescore as re; re.generate_features(...)  # annotate peaks: 20/40/0.35 (FTMS/TOF/ITMS), 'da'/'ppm', HCD
```

## Evaluation signals

- On the C*12:03 HLA dataset, rescoring must uniquely identify ~2574 peptides and ~14,318 PSMs, matching reported Figure 2C-D results (~489% peptide and ~296% PSM gains)
- Losses versus the native search engine score must stay minimal (~7 peptides and ~49 PSMs)
- Peak annotation uses the analyzer-specific default tolerances: 20 (FTMS), 40 (TOF), 0.35 (ITMS) with the matching unitMassTolerance ('da' or 'ppm')
- FDR estimation step defaults to fdr_estimation_method 'mokapot' and yields target/decoy-separated PSM and peptide level FDR
- Annotation only succeeds if RAW spectra were properly converted to mzML (ThermoRawFileParser, spectra_type 'raw') and loaded via pp.load_spectra

## Limitations

- Fragment mass tolerance and unitMassTolerance must be chosen per mass analyzer (FTMS/TOF/ITMS); wrong tolerance settings can mis-annotate peaks
- FDR estimation supports only 'percolator' or 'mokapot' as methods
- No discussion/changelog section content was available, so edge-case behavior beyond the documented defaults is not covered here

## Evidence

- [methods] Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [abstract] ML and deep learning (DL) models for peptide property prediction such as Prosit have enabled the creation of high quality in silico reference libraries: [evidence span withheld — anchor status: not_found]
- [readme] Oktoberfest is a python tool for collision energy calibration, rescoring search results and generating spectral libraries for proteomics research within the Prosit ecosystem.: [evidence span withheld — anchor status: external_doc]
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
