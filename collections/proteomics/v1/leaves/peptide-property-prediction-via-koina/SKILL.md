---
name: peptide-property-prediction-via-koina
description: Use when search engine PSM candidates lack Prosit-predicted properties
  and you need iRT or fragment-intensity predictions to compute rescoring features
  such as ΔRT or to build in-silico spectral libraries.
when_to_use_negative:
- The rescoring feature table already contains ΔRT/aligned-RT features from a prior
  prediction round — re-prediction would duplicate work
- No Koina server access is available (offline environment), since all predictions
  are retrieved by interfacing the server rather than computed locally
- The goal is purely FDR estimation on existing features with mokapot/percolator,
  with no new Prosit-predicted properties required
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Prosit
  role: Deep-learning model whose predicted peptide properties (fragment intensities,
    iRT) are retrieved as in-silico reference values
- name: Koina
  role: Prediction server that Oktoberfest interfaces to retrieve Prosit model predictions
    for PSM candidates
- name: Oktoberfest
  role: Provides pr.Predictor (high-level prediction runner) and re.generate_features
    to turn iRT predictions into the ΔRT rescoring feature
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: spectrum_fundamentals
  role: Converts search engine-specific modstrings to the ProForma standard so peptides
    are in the format Prosit/Koina expect
  repo: https://github.com/wilhelm-lab/spectrum_fundamentals
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:05ca3155ccdca677be1e995f5a17c7e681cc759be5679418d12351d512bbc31e
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# peptide-property-prediction-via-koina

## Summary

Retrieve Prosit deep-learning predictions of peptide properties (fragment intensities, iRT) for PSM candidates by interfacing a Koina prediction server from within Oktoberfest, providing the in-silico values needed for ΔRT rescoring features and spectral libraries.

## When to use

Use when search engine PSM candidates lack Prosit-predicted properties and you need iRT or fragment-intensity predictions to compute rescoring features such as ΔRT or to build in-silico spectral libraries.

## When NOT to use

- The rescoring feature table already contains ΔRT/aligned-RT features from a prior prediction round — re-prediction would duplicate work
- No Koina server access is available (offline environment), since all predictions are retrieved by interfacing the server rather than computed locally
- The goal is purely FDR estimation on existing features with mokapot/percolator, with no new Prosit-predicted properties required

## Inputs

- Search engine results in Oktoberfest internal format loaded via pp.load_search (confident target PSMs from the rescoring feature table)
- Peptide sequences with modstrings (converted to the ProForma standard by spectrum_fundamentals)
- Koina prediction server endpoint (koina.wilhelmlab.org:443, SSL) hosting Prosit models

## Outputs

- Per-PSM Prosit-predicted iRT values for the confident target PSM set
- Prosit-predicted fragment intensities usable for spectral library generation or spectral-angle rescoring features
- deltart_features.tsv containing observed RT, predicted iRT, and the computed ΔRT feature column

## How to apply

Load search engine results into Oktoberfest's internal format with oktoberfest.pp.load_search, selecting the confident target PSMs from the rescoring feature table (e.g. after pp.convert_search/pp.split_search). Use oktoberfest.pr.Predictor, the high-level prediction runner, to query the Koina prediction server at koina.wilhelmlab.org:443 over SSL, requesting the appropriate Prosit model (e.g. Prosit 2019 intensity, or Prosit-TMT/Prosit-timsTOF variants matching your acquisition) for iRT and fragment-intensity properties. The returned per-peptide iRT predictions are then mapped through an RT alignment fit — in re.generate_features with regressionMethod set to "lowess" (the config default is "spline", so lowess must be selected explicitly) — to compute ΔRT as the difference between predicted and observed retention time. Rationale: Prosit-quality in-silico reference values are what let the rescoring stage (mokapot or percolator) separate correct from incorrect PSMs.

## Related tools

- **Prosit** (Deep-learning model whose predicted peptide properties (fragment intensities, iRT) are retrieved as in-silico reference values)
- **Koina** (Prediction server that Oktoberfest interfaces to retrieve Prosit model predictions for PSM candidates)
- **Oktoberfest** (Provides pr.Predictor (high-level prediction runner) and re.generate_features to turn iRT predictions into the ΔRT rescoring feature) — https://github.com/wilhelm-lab/oktoberfest
- **spectrum_fundamentals** (Converts search engine-specific modstrings to the ProForma standard so peptides are in the format Prosit/Koina expect) — https://github.com/wilhelm-lab/spectrum_fundamentals

## Examples

```
from oktoberfest import pr; predictor = pr.Predictor(model='Prosit_2019_intensity', server='koina.wilhelmlab.org:443', ssl=True); predictor.predict(peptides)
```

## Evaluation signals

- deltart_features.tsv exists, is parseable, and contains fields for observed RT, predicted iRT, and the computed ΔRT feature for the confident PSM set
- Predictions were retrieved over SSL from koina.wilhelmlab.org:443 (network call logged / no connection errors)
- The regressionMethod used for the iRT-to-RT alignment matches intent: "lowess" was explicitly set, since the pipeline default is "spline"
- The number of predicted iRT values equals the number of confident target PSMs submitted from the loaded search results
- Downstream rescoring (mokapot or percolator) runs successfully on the feature table including the ΔRT column

## Limitations

- The config's regressionMethod default is "spline"; users wanting LOWESS alignment must explicitly select "lowess", otherwise ΔRT is computed with a different fit
- Prediction requires network access to a Koina server; SSL configuration on the non-default port 443 must be correct or retrieval fails
- The Prosit model variant must match the acquisition (Prosit-TMT for TMT-labeled peptides, Prosit-timsTOF for timsTOF), or predictions will not be appropriate

## Evidence

- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [abstract] ML and deep learning (DL) models for peptide property prediction such as Prosit have enabled the creation of high quality in silico reference libraries: [evidence span withheld — anchor status: not_found]
- [readme] It offers an end to end pipeline that takes search results, predicts peptide properties using koina, plots summaries and quality control figures and performs FDR estimation with either mokapot or percolator.: [evidence span withheld — anchor status: not_found]
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
