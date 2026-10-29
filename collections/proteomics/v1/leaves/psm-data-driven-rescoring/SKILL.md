---
name: psm-data-driven-rescoring
description: Use when MaxQuant (or similar) search results exist but identifications
  collapse on large search spaces, and Prosit-predicted fragment intensities plus
  mokapot FDR estimation could recover more PSMs at the same FDR.
when_to_use_negative:
- Input is not search engine output but already-rescored results or a feature table
  — re-running pp.convert_search on it is redundant
- The goal is spectral library generation or collision energy calibration rather than
  rescoring — configure a 'SpectralLibraryGeneration' or 'CollisionEnergCalibration'
  job type instead
- No access to a prediction server (e.g. Koina) is available, so Prosit fragment-intensity
  predictions required for feature generation cannot be retrieved
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: 'End-to-end rescoring pipeline: converts search results, generates features,
    and runs mokapot FDR estimation'
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model providing in silico fragment-intensity and iRT predictions
    used as rescoring features
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, ssl) interfaced by pr.Predictor
    to retrieve Prosit predictions
- name: mokapot
  role: Performs PSM-level target/decoy FDR estimation on the generated rescoring
    features (re.rescore_with_mokapot)
- name: Python
  role: Language in which Oktoberfest is implemented and invoked (Python 3.10+)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:682fc8b272ee58d84c8de0d509adcfe9a6ab83d8aab5e053e0ec0eab5a22edb1
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
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# psm-data-driven-rescoring

## Summary

Data-driven rescoring of search engine PSM results using Prosit-predicted spectral features via the Oktoberfest pipeline, boosting PSM identifications over native MaxQuant scores, especially as search-space size grows (e.g. metaproteomics against Swiss-Prot + IGC).

## When to use

Use when MaxQuant (or similar) search results exist but identifications collapse on large search spaces, and Prosit-predicted fragment intensities plus mokapot FDR estimation could recover more PSMs at the same FDR.

## When NOT to use

- Input is not search engine output but already-rescored results or a feature table — re-running pp.convert_search on it is redundant
- The goal is spectral library generation or collision energy calibration rather than rescoring — configure a 'SpectralLibraryGeneration' or 'CollisionEnergCalibration' job type instead
- No access to a prediction server (e.g. Koina) is available, so Prosit fragment-intensity predictions required for feature generation cannot be retrieved

## Inputs

- MaxQuant search engine output files (per search space: Swiss-Prot Human, +Bacteria, +IGC)
- Raw mass spectrometry spectra (spectra_type 'raw', e.g. PXD010871 from PRIDE)
- Fasta protein databases defining the search spaces

## Outputs

- Rescored PSM lists with mokapot target/decoy FDR estimates per search space
- Tabulated MaxQuant-only vs rescoring PSM counts (Figure 3C-D comparison: MaxQuant 11,724 -> 434 PSMs; rescoring 19,124 -> 114,510 PSMs)
- Quality-control figure psm_rescoring_comparison.png generated with oktoberfest.pl plotting functions

## How to apply

Download the search engine results and spectra (e.g. PXD010871 from PRIDE) and configure an Oktoberfest 'Rescoring' job with search_results_type 'Maxquant', spectra_type 'raw', and fdr_estimation_method 'mokapot'. Convert each search space's MaxQuant output to the internal format with pp.convert_search and pp.split_search, then prepare peptides for prediction using pp.filter_peptides_for_model. Retrieve Prosit fragment-intensity and iRT predictions via pr.Predictor interfacing the Koina server (prediction_server 'koina.wilhelmlab.org:443', ssl true). Annotate observed peaks and build rescoring features with re.generate_features, using massTolerance 20 ppm for FTMS and a 1.2 Da precursor peak-removal window, then rescore with re.rescore_with_mokapot at PSM-level target/decoy FDR. The rationale is that Prosit-derived in silico reference library features add information absent from native scores, so mokapot rescoring recovers identifications that MaxQuant loses when the search space grows.

## Related tools

- **Oktoberfest** (End-to-end rescoring pipeline: converts search results, generates features, and runs mokapot FDR estimation) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model providing in silico fragment-intensity and iRT predictions used as rescoring features)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, ssl) interfaced by pr.Predictor to retrieve Prosit predictions)
- **mokapot** (Performs PSM-level target/decoy FDR estimation on the generated rescoring features (re.rescore_with_mokapot))
- **Python** (Language in which Oktoberfest is implemented and invoked (Python 3.10+))

## Examples

```
import oktoberfest as ok; ok.re.rescore_with_mokapot(...)  # after ok.pp.convert_search, ok.pp.filter_peptides_for_model, ok.pr.Predictor(prediction_server='koina.wilhelmlab.org:443', ssl=True), ok.re.generate_features(...)
```

## Evaluation signals

- PSM counts at PSM-level 1% FDR show the reported pattern: MaxQuant PSMs collapse across the three search spaces (11,724 -> 434 on Swissprot+IGC) while rescoring retains substantially more (19,124 -> 114,510 PSMs)
- On the largest search space (Swissprot+IGC), rescoring retains ~81% (15,573 of 19,124) of human-only reference identifications vs 3.7% for MaxQuant
- The output figure psm_rescoring_comparison.png exists and reproduces the Figure 3C-D comparison
- re.rescore_with_mokapot completes FDR estimation on PSM level for each of the three search spaces without errors
- Peak annotation used massTolerance 20 (ppm) for FTMS data and a 1.2 Da precursor peak-removal window as configured

## Limitations

- Requires access to a Koina server for Prosit predictions; no local fallback is described in the provided context
- Raw spectra must be provided (spectra_type 'raw'); analysis starting from already-processed spectra is not covered
- Rescoring gains are demonstrated for MaxQuant search results and this metaproteomics use case; behavior with other search engines or extremely large search spaces beyond IGC is not established here
- FDR estimation method must be selected ('percolator' or 'mokapot'); using the wrong one changes result comparability

## Evidence

- [results] On the metaproteomics dataset with the largest search space (Swissprot+IGC), MaxQuant identified only 434 PSMs while data-driven rescoring identified 114,510 PSMs: [evidence span withheld — anchor status: not_found]
- [methods] The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring": "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [readme] It offers an end to end pipeline that takes search results, predicts peptide properties using koina, plots summaries and quality control figures and performs FDR estimation with either mokapot or percolator.: [evidence span withheld — anchor status: external_doc]
- [abstract] ML and deep learning (DL) models for peptide property prediction such as Prosit have enabled the creation of high quality in silico reference libraries: [evidence span withheld — anchor status: not_found]
