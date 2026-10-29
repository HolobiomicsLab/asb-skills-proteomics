---
name: public-proteomics-dataset-reanalysis
description: Use when a published proteomics dataset (e.g. PRIDE PXD010871 MaxQuant
  search results with raw spectra) must be re-processed to reproduce reported PSM
  counts or test data-driven rescoring against native search engine scores across
  search spaces.
when_to_use_negative:
- The dataset has no deposited search engine output — use Oktoberfest's spectral library
  generation or collision energy calibration workflows instead of rescoring
- You only need FDR estimation on an already-annotated feature table without Prosit
  prediction features — call mokapot directly rather than the full reanalysis pipeline
- The input is DIA data where the MaxQuant rescoring conversion path (pp.convert_search
  with search_results_type 'Maxquant') does not apply
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Oktoberfest
  role: 'End-to-end reanalysis pipeline: converts search results, generates Prosit-based
    rescoring features, and performs FDR estimation'
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model supplying in-silico fragment-intensity and iRT predictions
    for rescoring features
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443) interfaced via pr.Predictor to
    retrieve Prosit predictions
- name: mokapot
  role: PSM-level target/decoy FDR estimation during rescoring (re.rescore_with_mokapot)
- name: Python
  role: Execution environment (Python 3.10+) for running Oktoberfest as a package
    (import oktoberfest as ok)
  repo: https://pypi.org/project/oktoberfest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:f97b5f1243d2b0f36989059b7cb27d4ff009458adcfc2d56f76762856f7f7d09
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# public-proteomics-dataset-reanalysis

## Summary

Reanalyzes publicly deposited proteomics search results (e.g. PRIDE PXD010871 MaxQuant output) with Oktoberfest to reproduce or extend published rescoring findings, such as recovering PSMs lost in large metaproteomics search spaces via Prosit/Koina feature generation and mokapot FDR estimation.

## When to use

Use when a published proteomics dataset (e.g. PRIDE PXD010871 MaxQuant search results with raw spectra) must be re-processed to reproduce reported PSM counts or test data-driven rescoring against native search engine scores across search spaces.

## When NOT to use

- The dataset has no deposited search engine output — use Oktoberfest's spectral library generation or collision energy calibration workflows instead of rescoring
- You only need FDR estimation on an already-annotated feature table without Prosit prediction features — call mokapot directly rather than the full reanalysis pipeline
- The input is DIA data where the MaxQuant rescoring conversion path (pp.convert_search with search_results_type 'Maxquant') does not apply

## Inputs

- Publicly deposited search engine results (MaxQuant output from PRIDE PXD010871) for each search space (Swiss-Prot Human, +Bacteria, +IGC)
- Raw mass spectrometry spectra (.raw) associated with the deposited dataset
- Protein sequence databases / FASTA defining the search spaces

## Outputs

- Rescored PSM tables with mokapot-estimated PSM-level target/decoy FDR per search space
- Tabulated MaxQuant-only vs. rescoring PSM counts per search space
- Comparison figure (psm_rescoring_comparison.png) reproducing the MaxQuant-collapse / rescoring-recovery pattern

## How to apply

Download the public search results and spectra from PRIDE and configure an Oktoberfest 'Rescoring' job with search_results_type 'Maxquant', spectra_type 'raw', and fdr_estimation_method 'mokapot'. Convert each search engine output to Oktoberfest's internal format with pp.convert_search and pp.split_search, then prepare peptides with pp.filter_peptides_for_model and retrieve Prosit fragment-intensity and iRT predictions via pr.Predictor against the Koina server (koina.wilhelmlab.org:443, SSL enabled). Annotate observed peaks and build rescoring features with re.generate_features using massTolerance 20 ppm for FTMS and a 1.2 Da precursor peak-removal window (p_window). Rescore at PSM-level target/decoy FDR with re.rescore_with_mokapot, then tabulate and plot MaxQuant-only versus rescoring PSM counts per search space (e.g. Figure 3C-D: MaxQuant collapsing 11,724 to 434 PSMs; rescoring recovering 19,124 to 114,510 PSMs) and save the comparison figure.

## Related tools

- **Oktoberfest** (End-to-end reanalysis pipeline: converts search results, generates Prosit-based rescoring features, and performs FDR estimation) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model supplying in-silico fragment-intensity and iRT predictions for rescoring features)
- **Koina** (Prediction server (koina.wilhelmlab.org:443) interfaced via pr.Predictor to retrieve Prosit predictions)
- **mokapot** (PSM-level target/decoy FDR estimation during rescoring (re.rescore_with_mokapot))
- **Python** (Execution environment (Python 3.10+) for running Oktoberfest as a package (import oktoberfest as ok)) — https://pypi.org/project/oktoberfest/

## Examples

```
import oktoberfest as ok  # ok.pp.convert_search(...); ok.re.generate_features(...); ok.re.rescore_with_mokapot(...)
```

## Evaluation signals

- Reproduced PSM counts show MaxQuant collapse across the three search spaces (11,724 to 434 on Swissprot+IGC) while rescoring recovers to substantially higher counts (19,124 to 114,510)
- Rescoring retains ~81% (15,573 of 19,124) of human-only reference identifications versus 3.7% for MaxQuant on the largest search space
- Peak annotation tolerance settings applied correctly: massTolerance 20 ppm for FTMS (unitMassTolerance 'ppm') and p_window 1.2 Da precursor removal
- Output comparison figure psm_rescoring_comparison.png exists and matches the reported Figure 3C-D pattern
- mokapot rescored outputs report PSM-level target/decoy FDR estimates consistent with the configured fdr_estimation_method

## Limitations

- FDR estimation method must be one of 'percolator' or 'mokapot'; other estimators are not supported by the configuration
- Decoy generation defaults (specialAas 'KR') and prediction-server availability (Koina SSL endpoint) constrain reproducibility if server access changes
- Requires Python 3.10+ and Poetry for installation; raw-to-processed conversion depends on the spectrum_io/spectrum_fundamentals ecosystem

## Evidence

- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] Koina server for predictions: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [methods] Peak annotation tolerance: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [readme] README pipeline description: [evidence span withheld — anchor status: external_doc]
- [abstract] Reproduction of published results: "We demonstrate its ability to reproduce and even improve our results from previously published rescoring analyses on two distinct use cases."
