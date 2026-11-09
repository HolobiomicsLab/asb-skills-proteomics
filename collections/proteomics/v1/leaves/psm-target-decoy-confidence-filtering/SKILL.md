---
name: psm-target-decoy-confidence-filtering
description: Use when raw search engine PSM results (e.g. from Mascot, MSFragger or
  similar) must be converted to an internal format, split into target and decoy hits,
  filtered to a confident target set, and FDR-estimated with mokapot or percolator
  before deep-learning rescoring.
when_to_use_negative:
- Input is already a final mokapot/percolator-scored results table — filtering has
  already been performed and the confidence feature generation stage is unnecessary
- The goal is spectral library generation or collision energy calibration (job types
  "SpectralLibraryGeneration" or "CollisionEnergCalibration") rather than PSM rescoring
- The PSMs come from a search engine that Oktoberfest's pp.convert_search does not
  support
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Provides the pipeline functions (pp.convert_search, pp.load_search, pp.split_search,
    pp.filter_peptides, re.generate_features, re.rescore_with_mokapot/re.rescore_with_percolator)
    that convert search results, filter confident target PSMs, compute ΔRT features,
    and perform FDR estimation
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Target/decoy separation and FDR estimation on PSM and peptide level, invoked
    via re.rescore_with_mokapot as one of the two supported FDR backends
- name: Koina
  role: Prediction server that returns Prosit iRT predictions for the confident target
    PSMs used in the ΔRT feature calculation
- name: Prosit
  role: Deep learning model supplying peptide property predictions (iRT) that are
    aligned to observed RT for the delta-RT rescoring feature
- name: Python
  role: Oktoberfest is an open source Python package (Python 3.10+) used programmatically
    for the filtering and rescoring steps
  repo: https://pypi.org/project/oktoberfest/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:bb8aba4dcbd7b200e9fb62f668b2857e8b77a0a89358047759dc64eff03709f1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# psm-target-decoy-confidence-filtering

## Summary

Selects confident target PSMs from search engine results and estimates PSM/peptide-level confidence via target-decoy FDR estimation, using Oktoberfest's conversion of search output into an internal feature table that feeds Prosit/Koina-based rescoring features such as ΔRT.

## When to use

Use when raw search engine PSM results (e.g. from Mascot, MSFragger or similar) must be converted to an internal format, split into target and decoy hits, filtered to a confident target set, and FDR-estimated with mokapot or percolator before deep-learning rescoring.

## When NOT to use

- Input is already a final mokapot/percolator-scored results table — filtering has already been performed and the confidence feature generation stage is unnecessary
- The goal is spectral library generation or collision energy calibration (job types "SpectralLibraryGeneration" or "CollisionEnergCalibration") rather than PSM rescoring
- The PSMs come from a search engine that Oktoberfest's pp.convert_search does not support

## Inputs

- Search engine result files (e.g. mzIdentML or engine-specific PSM outputs) convertible by pp.convert_search
- Rescoring feature table with target and decoy PSMs in Oktoberfest internal format
- Oktoberfest job config specifying the FDR estimation method ("percolator" or "mokapot")
- Prosit-predicted iRT values for confident target PSMs from the Koina server (koina.wilhelmlab.org:443, SSL)

## Outputs

- Confident target PSM subset of the internal-format search results
- Rescoring feature table augmented with per-PSM features (e.g. deltart_features.tsv containing observed RT, predicted iRT, and delta-RT columns)
- FDR-estimated PSM/peptide identifications after re.rescore_with_mokapot or re.rescore_with_percolator

## How to apply

Load the search engine output into Oktoberfest's internal format with pp.load_search (after pp.convert_search and pp.split_search), which yields the rescoring feature table with target and decoy PSMs. Filter peptides with pp.filter_peptides / pp.filter_peptides_for_model to retain a confident target PSM set suitable for property prediction. Estimate target/decoy separation and FDR on PSM and peptide level by choosing "percolator" or "mokapot" in the job configuration; mokapot is invoked through re.rescore_with_mokapot (analogously re.rescore_with_percolator). The confident PSM subset then serves as input for downstream features — e.g. Prosit iRT predictions retrieved via Koina are LOWESS-fitted to observed RT (regressionMethod "lowess", alternative "spline" or "logistic") to compute ΔRT per PSM in deltart_features.tsv.

## Related tools

- **Oktoberfest** (Provides the pipeline functions (pp.convert_search, pp.load_search, pp.split_search, pp.filter_peptides, re.generate_features, re.rescore_with_mokapot/re.rescore_with_percolator) that convert search results, filter confident target PSMs, compute ΔRT features, and perform FDR estimation) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Target/decoy separation and FDR estimation on PSM and peptide level, invoked via re.rescore_with_mokapot as one of the two supported FDR backends)
- **Koina** (Prediction server that returns Prosit iRT predictions for the confident target PSMs used in the ΔRT feature calculation)
- **Prosit** (Deep learning model supplying peptide property predictions (iRT) that are aligned to observed RT for the delta-RT rescoring feature)
- **Python** (Oktoberfest is an open source Python package (Python 3.10+) used programmatically for the filtering and rescoring steps) — https://pypi.org/project/oktoberfest/

## Examples

```
regressionMethod = "lowess"  # Oktoberfest rescoring config: "spline" | "lowess" | "logistic"; default is "spline"
```

## Evaluation signals

- deltart_features.tsv exists, is a parseable table, and contains fields for observed RT, predicted iRT, and the computed delta-RT feature for the confident PSM set
- Delta-RT values are present for every PSM in the confident target set and reflect the deviation between predicted (Prosit iRT mapped through the LOWESS fit) and experimental retention time
- The regression method actually used is the one configured — "lowess" must be explicitly selected because the pipeline default is "spline"
- FDR estimation produced results at both PSM and peptide level using the configured method (percolator or mokapot), with target and decoy PSMs both represented in the internal feature table
- The final mokapot-rescored output (from re.rescore_with_mokapot) covers the same PSMs that entered feature generation, confirming no PSMs were silently dropped between steps

## Limitations

- The regressionMethod config default is "spline", so a LOWESS-based ΔRT calculation must be explicitly requested; using the default silently changes the fitted curve
- Delta-RT feature quality depends on access to a Koina prediction server (koina.wilhelmlab.org:443 with SSL) and on Prosit covering the peptide chemistries in the search results
- FDR estimation supports only the two configured backends, "percolator" or "mokapot"; other tools are not wired into the pipeline

## Evidence

- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] re.generate_features is listed among the rescoring functions in the API surface: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
- [readme] README describes the end-to-end pipeline including FDR estimation with mokapot or percolator: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract positions Oktoberfest as the open source package for the rescoring pipeline: "we present Oktoberfest, an open source Python package of our spectral library generation and rescoring pipeline"
