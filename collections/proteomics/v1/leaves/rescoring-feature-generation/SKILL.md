---
name: rescoring-feature-generation
description: Use when a Rescoring-type Oktoberfest job needs supplementary features
  beyond raw search scores — e.g. PSM candidates from a search engine must be rescored
  with a ΔRT feature computed from Prosit iRT predictions vs. observed RT.
when_to_use_negative:
- The input is already a completed rescoring feature table with ΔRT values (proceed
  directly to re.rescore_with_mokapot instead)
- The job is a SpectralLibraryGeneration or CollisionEnergCalibration task rather
  than Rescoring
- No Prosit/Koina-compatible iRT predictions are available for the peptide set (e.g.
  non-standard modifications outside UniMod mapping at unimod.org)
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Oktoberfest
  role: 'Python package executing the rescoring pipeline: pp.load_search, re.generate_features
    (regressionMethod lowess/spline/logistic) and deltart feature writing'
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model providing the iRT (peptide property) predictions that
    are aligned to observed RT
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, SSL) queried via oktoberfest.pr.Predictor
    to retrieve Prosit iRT predictions
- name: mokapot
  role: Consumes the feature table (including ΔRT) for target/decoy separation and
    FDR estimation on PSM and peptide level via re.rescore_with_mokapot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:90677a298b735e807ee67c57a7b1e2b3847a94d256fbb5ba39707271635e6a86
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rescoring-feature-generation

## Summary

Generates data-driven rescoring features (notably ΔRT) for PSMs in the Oktoberfest pipeline by aligning Prosit-predicted iRT values to experimentally observed retention times with a LOWESS fit, then writing the per-PSM delta-RT column into the feature table used by mokapot/percolator FDR estimation.

## When to use

Use when a Rescoring-type Oktoberfest job needs supplementary features beyond raw search scores — e.g. PSM candidates from a search engine must be rescored with a ΔRT feature computed from Prosit iRT predictions vs. observed RT.

## When NOT to use

- The input is already a completed rescoring feature table with ΔRT values (proceed directly to re.rescore_with_mokapot instead)
- The job is a SpectralLibraryGeneration or CollisionEnergCalibration task rather than Rescoring
- No Prosit/Koina-compatible iRT predictions are available for the peptide set (e.g. non-standard modifications outside UniMod mapping at unimod.org)

## Inputs

- Search engine results in Oktoberfest internal format (via pp.convert_search / pp.load_search)
- Confident target PSM candidates from the rescoring feature table
- Prosit-predicted iRT values for the confident peptides (from Koina)
- Oktoberfest rescoring config (jobType 'Rescoring') with regressionMethod setting

## Outputs

- deltart_features.tsv containing observed RT, predicted iRT, and computed delta-RT per PSM
- Rescoring feature table enriched with the ΔRT column for downstream mokapot/percolator FDR estimation

## How to apply

First load the search engine results with oktoberfest.pp.load_search (after pp.convert_search / pp.split_search) and select the confident target PSMs from the rescoring feature table. Retrieve Prosit-predicted iRT values for those peptides through the Koina interface (oktoberfest.pr.Predictor against koina.wilhelmlab.org:443 with SSL), as Oktoberfest 'interfac[es] a Koina server to retrieve predictions from various prediction models'. Fit the iRT-to-observed-RT curve with re.generate_features using the regressionMethod option set to 'lowess' — the config supports 'spline', 'lowess', or 'logistic' but the pipeline default is 'spline', so lowess must be explicitly selected. Map each PSM's predicted iRT through the fitted curve and compute ΔRT as the deviation between predicted and experimental retention time, then write the values as a per-PSM feature column into deltart_features.tsv which feeds re.rescore_with_mokapot or re.rescore_with_percolator for target/decoy separation and FDR estimation on PSM and peptide level.

## Related tools

- **Oktoberfest** (Python package executing the rescoring pipeline: pp.load_search, re.generate_features (regressionMethod lowess/spline/logistic) and deltart feature writing) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model providing the iRT (peptide property) predictions that are aligned to observed RT)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, SSL) queried via oktoberfest.pr.Predictor to retrieve Prosit iRT predictions)
- **mokapot** (Consumes the feature table (including ΔRT) for target/decoy separation and FDR estimation on PSM and peptide level via re.rescore_with_mokapot)

## Examples

```
re.generate_features(results, output="deltart_features.tsv", regressionMethod="lowess")
```

## Evaluation signals

- deltart_features.tsv exists in the output directory and parses as a table
- The table contains fields for observed RT, predicted iRT, and the computed delta-RT feature for the confident PSM set
- The regressionMethod used is 'lowess' as explicitly configured, since the pipeline default is 'spline'
- ΔRT is computed as the deviation between predicted (iRT-mapped) and experimental retention time for every confident target PSM
- Downstream mokapot/percolator rescoring accepts the feature table and performs PSM- and peptide-level FDR estimation without schema errors

## Limitations

- The pipeline default regressionMethod is 'spline', so forgetting to explicitly set regressionMethod='lowess' silently yields a different alignment model
- ΔRT computation requires network access to the Koina prediction server for Prosit iRT predictions
- Modification masses must map to UniMod IDs (retrievable from unimod.org), otherwise Prosit prediction of the peptide may fail

## Evidence

- [readme] Oktoberfest is a python tool for collision energy calibration, rescoring search results and generating spectral libraries for proteomics research within the Prosit ecosystem.: [evidence span withheld — anchor status: external_doc]
- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [readme] It offers an end to end pipeline that takes search results, predicts peptide properties using koina, plots summaries and quality control figures and performs FDR estimation with either mokapot or percolator.: [evidence span withheld — anchor status: external_doc]
