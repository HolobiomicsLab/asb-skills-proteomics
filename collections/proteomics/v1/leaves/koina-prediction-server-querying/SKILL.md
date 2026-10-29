---
name: koina-prediction-server-querying
description: Use when peptides (from a .csv precursor list or in-silico-digested FASTA)
  need predicted fragment intensities or iRT values to build a Spectronaut-compatible
  .csv/.msp spectral library or generate rescoring features.
when_to_use_negative:
- Experimental spectra already exist for the peptides of interest and an empirical
  spectral library can be built instead of an in-silico one
- The task is raw-file conversion or search-result parsing (pp.convert_raw_to_mzml,
  pp.convert_search) rather than peptide property prediction
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Koina
  role: Prediction server hosting Prosit-family models from which fragment-intensity
    and iRT predictions are retrieved
- name: Oktoberfest
  role: Python package whose pr.Koina/pr.Predictor interface wraps server queries
    and integrates predictions into library generation and rescoring
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep-learning model family whose predictions are served through Koina for
    peptide property prediction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:adc5b5b42999a5f1ce1bea02a42dce475013401dbcea97e1dab52756fe65fc19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Prosit
  - Python
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Koina prediction server querying

## Summary

Queries a Koina server (default koina.wilhelmlab.org:443 over SSL) via Oktoberfest's pr.Koina/pr.Predictor interface to retrieve Prosit-family fragment-intensity and iRT predictions, which underpin in-silico spectral library generation and rescoring.

## When to use

Use when peptides (from a .csv precursor list or in-silico-digested FASTA) need predicted fragment intensities or iRT values to build a Spectronaut-compatible .csv/.msp spectral library or generate rescoring features.

## When NOT to use

- Experimental spectra already exist for the peptides of interest and an empirical spectral library can be built instead of an in-silico one
- The task is raw-file conversion or search-result parsing (pp.convert_raw_to_mzml, pp.convert_search) rather than peptide property prediction

## Inputs

- .csv file of requested precursor peptides or peptides from in-silico digestion of a proteome .fasta (pp.digest with trypsin, full digestion, 2 missed cleavages, lengths 7-60)
- Oktoberfest configuration file specifying intensity and iRT model names, fragmentation method, and collision energy

## Outputs

- Predicted fragment-intensity vectors and iRT values for each precursor
- Annotated spectra feeding Spectronaut-compatible .csv and .msp spectral libraries (via pp.gen_lib)

## How to apply

Configure an Oktoberfest job (e.g. type "SpectralLibraryGeneration") naming the intensity and iRT Prosit models to use, then retrieve predictions through pr.Koina/pr.Predictor against the Koina server (default koina.wilhelmlab.org:443, ssl true). During prediction apply library options such as fragmentation "HCD" or "CID", collision energy 30 (default), precursor charges [2,3], at most 1 methionine oxidation (nrOx = 1), a minimal relative intensity threshold of 5e-4, and a prediction batch size of 10000 peptides. Predictions are then merged with digested peptides (pp.merge_spectra_and_peptides) and annotated (pp.annotate_spectral_library) before writing the library with pp.gen_lib. In rescoring workflows, the same Koina interface supplies predicted properties used to generate features for mokapot or percolator FDR estimation.

## Related tools

- **Koina** (Prediction server hosting Prosit-family models from which fragment-intensity and iRT predictions are retrieved)
- **Oktoberfest** (Python package whose pr.Koina/pr.Predictor interface wraps server queries and integrates predictions into library generation and rescoring) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep-learning model family whose predictions are served through Koina for peptide property prediction)

## Examples

```
import oktoberfest as ok; predictor = ok.pr.Predictor(config)  # retrieves intensity/iRT predictions from koina.wilhelmlab.org:443
```

## Evaluation signals

- Spectral library files (.csv and .msp) are present in the configured output folder in both requested formats
- Predicted fragments below the minimal relative intensity threshold of 5e-4 are excluded from the annotated library
- Batch processing completes without server errors at the configured batch size of 10000 peptides, using koina.wilhelmlab.org:443 with ssl true
- Precursor space matches configuration: precursor charges [2,3] and at most 1 methionine oxidation (nrOx = 1)

## Limitations

- Predictions require connectivity to the Koina server; the default endpoint koina.wilhelmlab.org:443 must be reachable over SSL
- Prediction scope is bounded by the configured model: fragmentation limited to "HCD" or "CID" at collision energy 30 (default), so non-tryptic or alternative-fragmentation use cases need the appropriate Prosit model (e.g. Prosit-timsTOF, Prosit-MultiFrag) and citation
- Model choice carries citation obligations: Prosit, Prosit-TMT, Prosit-timsTOF, Prosit-XL, and Prosit-MultiFrag each require citing their original publication per the README

## Evidence

- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [abstract] ML and deep learning models enable in silico libraries: [evidence span withheld — anchor status: not_found]
- [methods] specialAas default and decoy generation: "specialAas                 | Special amino acids for decoy generation; default = "KR""
- [readme] README tool description: [evidence span withheld — anchor status: external_doc]
