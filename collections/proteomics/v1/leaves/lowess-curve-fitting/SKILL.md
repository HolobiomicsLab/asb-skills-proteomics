---
name: lowess-curve-fitting
description: 'Use when search-engine PSMs need a retention-time-based rescoring feature:
  Prosit iRT predictions for confident target PSMs must be mapped through a LOWESS
  fit to observed RT so ΔRT deviations can separate correct from incorrect PSMs.'
when_to_use_negative:
- 'Do not assume LOWESS is active by default: the Oktoberfest regressionMethod config
  default is "spline", so a practitioner wanting LOWESS must explicitly set it; conversely,
  if the spline or logistic regression fit is desired, this skill''s specific LOWESS
  path is not the right configuration.'
- If the goal is spectral library generation or collision energy calibration (the
  other Oktoberfest job types) rather than ΔRT feature computation for rescoring,
  this skill does not apply.
- If no experimentally observed retention times are available (e.g. peptides without
  search-engine PSM hits), there is nothing to align against and ΔRT cannot be computed.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Python package providing re.generate_features, which fits the regression curve
    (LOWESS/spline/logistic) between predicted iRTs and observed RTs and generates
    the delta-RT feature table
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model whose predicted iRT values for target PSM candidates are
    the input side of the iRT-to-RT alignment
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, SSL) through which Prosit iRT
    predictions are retrieved via Oktoberfest's interfacing functions
- name: mokapot
  role: Consumes the delta-RT feature table downstream for target/decoy FDR estimation
    during rescoring (re.rescore_with_mokapot)
  repo: https://github.com/wilhelm-lab/oktoberfest
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:235de680b9fc671f9f7a9de2d9834b8645a73978100c7b4d7bc468803b301c2c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# lowess-curve-fitting

## Summary

Fits a locally weighted scatterplot smoothing (LOWESS) curve to align Prosit-predicted iRT values with experimentally observed retention times from search engine results, then computes per-PSM ΔRT (predicted minus observed RT) as a feature for data-driven PSM rescoring in the Oktoberfest pipeline.

## When to use

Use when search-engine PSMs need a retention-time-based rescoring feature: Prosit iRT predictions for confident target PSMs must be mapped through a LOWESS fit to observed RT so ΔRT deviations can separate correct from incorrect PSMs.

## When NOT to use

- Do not assume LOWESS is active by default: the Oktoberfest regressionMethod config default is "spline", so a practitioner wanting LOWESS must explicitly set it; conversely, if the spline or logistic regression fit is desired, this skill's specific LOWESS path is not the right configuration.
- If the goal is spectral library generation or collision energy calibration (the other Oktoberfest job types) rather than ΔRT feature computation for rescoring, this skill does not apply.
- If no experimentally observed retention times are available (e.g. peptides without search-engine PSM hits), there is nothing to align against and ΔRT cannot be computed.

## Inputs

- Search engine results in Oktoberfest internal format (loaded via pp.load_search from the rescoring feature table)
- Confident target PSM set selected from the search results
- Prosit-predicted iRT values per peptide retrieved from the Koina server (koina.wilhelmlab.org:443, SSL)

## Outputs

- deltart_features.tsv — a per-PSM rescoring feature table containing observed RT, predicted iRT, and the computed delta-RT column for the confident PSM set
- Fitted LOWESS iRT-to-observed-RT curve used to map predicted iRTs to expected RT

## How to apply

Load search engine results in Oktoberfest's internal format via oktoberfest.pp.load_search, retaining only confident target PSMs from the rescoring feature table. Retrieve Prosit-predicted iRT values for those PSMs through the Koina interface (prediction server koina.wilhelmlab.org:443 with SSL) rather than local inference. Fit the iRT-to-observed-RT curve by calling oktoberfest re.generate_features with the regressionMethod option explicitly set to "lowess" — the pipeline default is "spline", so LOWESS must be selected manually (the config also supports "logistic"). Map each PSM's predicted iRT through the fitted curve and compute ΔRT as the deviation between predicted and experimental retention time. Write the ΔRT values into deltart_features.tsv as a per-PSM feature column, then merge it with other rescoring features (e.g. via re.merge_input) before mokapot-based FDR estimation. The rationale is that LOWESS provides a non-parametric, robust local fit of the iRT-to-RT relationship, so residual deviations (ΔRT) expose peptides whose retention behavior disagrees with the Prosit prediction.

## Related tools

- **Oktoberfest** (Python package providing re.generate_features, which fits the regression curve (LOWESS/spline/logistic) between predicted iRTs and observed RTs and generates the delta-RT feature table) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model whose predicted iRT values for target PSM candidates are the input side of the iRT-to-RT alignment)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, SSL) through which Prosit iRT predictions are retrieved via Oktoberfest's interfacing functions)
- **mokapot** (Consumes the delta-RT feature table downstream for target/decoy FDR estimation during rescoring (re.rescore_with_mokapot)) — https://github.com/wilhelm-lab/oktoberfest

## Examples

```
from oktoberfest import re; re.generate_features(search_result_psm_list, output_path, regressionMethod="lowess")
```

## Evaluation signals

- deltart_features.tsv exists after the run and is a parseable table
- The table contains fields for observed RT, predicted iRT, and the computed delta-RT feature for the confident PSM set
- The regressionMethod in the config reads "lowess" (not the pipeline default "spline")
- ΔRT values for confident (correct) PSMs cluster near zero, since these PSMs were used to anchor the fit
- The ΔRT column is successfully merged into the mokapot rescoring feature table (re.merge_input) and consumed by re.rescore_with_mokapot

## Limitations

- LOWESS must be explicitly selected: the pipeline default regressionMethod is "spline", so omitting the setting silently yields a different fit.
- The ΔRT feature is only meaningful for the confident target PSM set used to fit the iRT-to-RT curve; extrapolation beyond that iRT range is unvalidated by the described workflow.
- Fitting depends on network access to the Koina prediction server (koina.wilhelmlab.org:443) to obtain Prosit iRT predictions.

## Evidence

- [methods] re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [readme] takes search results, predicts peptide properties using koina: [evidence span withheld — anchor status: external_doc]
- [other] Prosit-predicted iRTs for target PSM candidates are aligned to experimentally observed retention times using a locally weighted scatterplot smoothing (LOWESS) fit: [evidence span withheld — anchor status: not_found]
