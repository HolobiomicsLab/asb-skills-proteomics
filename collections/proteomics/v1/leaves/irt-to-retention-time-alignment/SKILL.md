---
name: irt-to-retention-time-alignment
description: 'Use when building rescoring features for PSM-level FDR estimation: you
  have search engine results with confident target PSMs, Prosit-predicted iRT values,
  and need a ΔRT (predicted vs. observed RT) feature column for mokapot/percolator
  rescoring.'
when_to_use_negative:
- The input is already a complete rescoring feature table that contains a ΔRT column
  — re-running the alignment would only duplicate the feature
- The goal is spectral library generation (pp.gen_lib) or collision energy calibration
  rather than data-driven rescoring of search engine results
- No Prosit/Koina iRT predictions are available or reachable for the PSMs, so the
  iRT-to-RT alignment cannot be fit
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Oktoberfest
  role: Provides pp.load_search to load search results and re.generate_features to
    fit the iRT-to-RT regression (regressionMethod='lowess') and write the ΔRT feature
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, SSL) from which Prosit-predicted
    iRT values for the confident PSM set are retrieved
- name: Prosit
  role: Deep learning model producing the peptide property (iRT) predictions that
    are aligned to observed retention times
- name: mokapot
  role: Consumes the ΔRT feature as part of downstream target/decoy separation and
    FDR estimation via re.rescore_with_mokapot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:0d968fdf2c1cb6c47e8c11a63ae04de39f885c05b326f0d276626c8709bb0600
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# irt-to-retention-time-alignment

## Summary

In Oktoberfest's data-driven rescoring pipeline, Prosit-predicted iRT values for confident target PSM candidates are aligned to experimentally observed retention times via a LOWESS fit, and the fit is used to compute a per-PSM ΔRT feature that separates correct from incorrect PSMs.

## When to use

Use when building rescoring features for PSM-level FDR estimation: you have search engine results with confident target PSMs, Prosit-predicted iRT values, and need a ΔRT (predicted vs. observed RT) feature column for mokapot/percolator rescoring.

## When NOT to use

- The input is already a complete rescoring feature table that contains a ΔRT column — re-running the alignment would only duplicate the feature
- The goal is spectral library generation (pp.gen_lib) or collision energy calibration rather than data-driven rescoring of search engine results
- No Prosit/Koina iRT predictions are available or reachable for the PSMs, so the iRT-to-RT alignment cannot be fit

## Inputs

- Search engine results in Oktoberfest internal format (loaded via oktoberfest.pp.load_search), with confident target PSMs selected from the rescoring feature table
- Prosit-predicted iRT values for the confident PSM set, retrieved from a Koina server (koina.wilhelmlab.org:443, SSL)

## Outputs

- Fitted iRT-to-observed-RT regression curve (LOWESS) mapping predicted iRT to experimental retention time
- deltart_features.tsv: per-PSM rescoring feature table with columns for observed RT, predicted iRT, and the computed ΔRT feature for the confident PSM set

## How to apply

Load the search engine results in internal format with oktoberfest.pp.load_search and select the confident target PSMs from the rescoring feature table. Retrieve Prosit-predicted iRT values for those PSMs through the Koina interface (prediction server koina.wilhelmlab.org:443 with SSL). Fit the iRT-to-observed-RT curve with oktoberfest re.generate_features, explicitly setting the regressionMethod option to 'lowess' (the config supports 'spline', 'lowess', or 'logistic', but the pipeline default is 'spline', so lowess must be selected explicitly). Map each PSM's predicted iRT through the fitted curve and compute delta-RT as the deviation between predicted and experimental retention time, writing the values as a per-PSM feature column into deltart_features.tsv. The rationale is that correct PSMs should show small deviation between the Prosit-predicted iRT (mapped through the LOWESS fit) and the experimentally observed RT, making ΔRT a discriminative feature for target/decoy separation during FDR estimation.

## Related tools

- **Oktoberfest** (Provides pp.load_search to load search results and re.generate_features to fit the iRT-to-RT regression (regressionMethod='lowess') and write the ΔRT feature) — https://github.com/wilhelm-lab/oktoberfest
- **Koina** (Prediction server (koina.wilhelmlab.org:443, SSL) from which Prosit-predicted iRT values for the confident PSM set are retrieved)
- **Prosit** (Deep learning model producing the peptide property (iRT) predictions that are aligned to observed retention times)
- **mokapot** (Consumes the ΔRT feature as part of downstream target/decoy separation and FDR estimation via re.rescore_with_mokapot)

## Examples

```
from oktoberfest import re; re.generate_features(..., regressionMethod="lowess")  # fit iRT→observed RT curve and write deltart_features.tsv
```

## Evaluation signals

- deltart_features.tsv exists and is a parseable table in the output directory
- deltart_features.tsv contains fields for observed RT, predicted iRT, and the computed delta-RT feature for the confident PSM set
- The fitted curve was generated with regressionMethod explicitly set to 'lowess' (not the pipeline default 'spline') in the re.generate_features config
- Each confident PSM in the input feature table has a non-missing ΔRT value after alignment
- Correct PSMs show small |ΔRT| (small deviation between predicted and experimental RT) relative to decoys, consistent with ΔRT's use in separating correct from incorrect PSMs

## Limitations

- The pipeline default regressionMethod is 'spline', so leaving the config unchanged silently produces a spline instead of the LOWESS alignment described here
- The regressionMethod config supports only 'spline', 'lowess', or 'logistic' — other regression strategies are not available
- Retrieval of Prosit iRT predictions requires network access to a Koina server (koina.wilhelmlab.org:443 with SSL), so the step fails without connectivity to the prediction server

## Evidence

- [abstract] used in various applications, ranging from data-independent acquisition (DIA) data analysis to data-driven rescoring of search engine results: "used in various applications, ranging from data-independent acquisition (DIA) data analysis to data-driven rescoring of search engine results"
- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [methods] re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator: "re.generate_features
   re.merge_input
   re.rescore_with_mokapot
   re.rescore_with_percolator"
- [readme] Oktoberfest is a python tool for collision energy calibration, rescoring search results and generating spectral libraries for proteomics research within the Prosit ecosystem.: [evidence span withheld — anchor status: external_doc]
