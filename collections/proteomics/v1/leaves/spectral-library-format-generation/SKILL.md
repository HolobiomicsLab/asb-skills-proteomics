---
name: spectral-library-format-generation
description: Use when a DIA/search workflow requires a predicted spectral library
  and the input is either a .csv of precursor peptides or a proteome .fasta needing
  in silico digestion into target-decoy libraries.
when_to_use_negative:
- The goal is rescoring search-engine results or collision energy calibration (job
  types 'Rescoring' or 'CollisionEnergCalibration'), not library generation.
- Empirical, measured spectral libraries already exist for the precursors of interest
  and prediction is unnecessary.
- The input is raw mass-spec data (e.g. .raw files needing pp.convert_raw_to_mzml)
  rather than a peptide list or proteome FASTA.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Open-source Python package whose pp.* functions (pp.digest, pp.generate_metadata,
    pp.merge_spectra_and_peptides, pp.annotate_spectral_library, pp.gen_lib) execute
    the library generation pipeline and write Spectronaut and .msp outputs
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep-learning model for peptide property (fragment intensity) prediction that
    underpins the in silico spectral library; accessed via Koina
- name: Koina
  role: Prediction server (default koina.wilhelmlab.org:443, ssl true) from which
    fragment-intensity and iRT predictions are retrieved via pr.Koina/pr.Predictor
- name: spectrum_fundamentals
  role: Supporting package within the Oktoberfest pipeline providing theoretical mass
    calculation and spectral annotation used during library assembly
  repo: https://github.com/wilhelm-lab/spectrum_fundamentals
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:148e38eec7277a4079c1619469c4b7cfc100877a40793c82bc79756c88f884c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Prosit
  - Python
  - spectrum_fundamentals
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# spectral-library-format-generation

## Summary

Generates in silico spectral libraries in Spectronaut-compatible .csv and .msp formats from a peptide .csv or proteome .fasta input, using Oktoberfest's digest→predict→annotate→write pipeline with Prosit/Koina predictions.

## When to use

Use when a DIA/search workflow requires a predicted spectral library and the input is either a .csv of precursor peptides or a proteome .fasta needing in silico digestion into target-decoy libraries.

## When NOT to use

- The goal is rescoring search-engine results or collision energy calibration (job types 'Rescoring' or 'CollisionEnergCalibration'), not library generation.
- Empirical, measured spectral libraries already exist for the precursors of interest and prediction is unnecessary.
- The input is raw mass-spec data (e.g. .raw files needing pp.convert_raw_to_mzml) rather than a peptide list or proteome FASTA.

## Inputs

- Proteome .fasta file (library_input with library_input_type='fasta') for in-silico digestion
- .csv file of requested precursor peptides (library_input with library_input_type='csv')
- Oktoberfest configuration file with type 'SpectralLibraryGeneration'
- Configured intensity model and irt model names (e.g. Prosit models served on Koina)

## Outputs

- Spectronaut-compatible .csv spectral library in the output folder
- .msp spectral library in the output folder

## How to apply

Create an Oktoberfest config with type 'SpectralLibraryGeneration' and set library_input plus library_input_type ('fasta' or 'csv'). If a FASTA is given, digest in silico with pp.digest using trypsin, full digestion, 2 missed cleavages, peptide lengths 7-60, specialAas 'KR', and a concatenated target-decoy database (db='concat'). Retrieve fragment-intensity and iRT predictions from the Koina server (koina.wilhelmlab.org:443, ssl true) via pr.Koina/pr.Predictor using configured intensity and irt models, applying library options such as fragmentation 'HCD' or 'CID', collision energy 30 (default), precursor charges [2,3], at most 1 methionine oxidation (nrOx=1), a minimal relative intensity threshold of 5e-4, and a prediction batch size of 10000 peptides. Assemble the library with pp.generate_metadata, pp.merge_spectra_and_peptides, and pp.annotate_spectral_library, then write it with pp.gen_lib in both 'spectronaut' and 'msp' formats to the configured output folder. The rationale is that Prosit-style deep-learning predictions enable high-quality in silico reference libraries without measuring empirical spectra for every precursor.

## Related tools

- **Oktoberfest** (Open-source Python package whose pp.* functions (pp.digest, pp.generate_metadata, pp.merge_spectra_and_peptides, pp.annotate_spectral_library, pp.gen_lib) execute the library generation pipeline and write Spectronaut and .msp outputs) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep-learning model for peptide property (fragment intensity) prediction that underpins the in silico spectral library; accessed via Koina)
- **Koina** (Prediction server (default koina.wilhelmlab.org:443, ssl true) from which fragment-intensity and iRT predictions are retrieved via pr.Koina/pr.Predictor)
- **spectrum_fundamentals** (Supporting package within the Oktoberfest pipeline providing theoretical mass calculation and spectral annotation used during library assembly) — https://github.com/wilhelm-lab/spectrum_fundamentals

## Examples

```
import oktoberfest as ok; ok.pp.digest(...); ok.pr.Predictor(...); ok.pp.annotate_spectral_library(...); ok.pp.gen_lib(..., format=['spectronaut','msp'])
```

## Evaluation signals

- Generated spectral library files are present in the configured output folder in both requested formats (Spectronaut-compatible .csv and .msp).
- In-silico digestion used the configured parameters: trypsin, full digestion, 2 missed cleavages, lengths 7-60, specialAas 'KR', and db='concat' target-decoy concatenation.
- Prediction settings were honored: precursor charges [2,3], collision energy 30 (default), nrOx = 1, minimal relative intensity 5e-4, and batch size 10000 peptides.
- The .csv output is Spectronaut-compatible (loadable by Spectronaut) and the .msp output parses as a standard spectral library.
- Koina retrieval succeeded against the default server koina.wilhelmlabs.org:443 with ssl true for both the intensity and rt models.

## Limitations

- Fragmentation options are limited to configured modes such as 'HCD' or 'CID'; other fragmentation techniques require different Prosit models (e.g. Prosit-timsTOF, Prosit-MultiFrag) cited separately in the README.
- Prediction batch size caps throughput at 10000 peptides per request; very large proteomes require chunked Koina retrieval.
- Discussion/changelog content is unavailable in the source, so long-term maintenance status and known bugs cannot be assessed from the article alone.

## Evidence

- [methods] The library generation workflow accepts either a .csv file of the requested precursor peptides or a .fasta file, which Oktoberfest digests and prepares in silico, and it outputs Spectronaut-compatible .csv and .msp spectral library files.: [evidence span withheld — anchor status: not_found]
- [methods] Digest the proteome in silico with pp.digest using trypsin, full digestion mode, 2 missed cleavages, peptide lengths 7-60, specialAas KR, and a concatenated target-decoy database.: [evidence span withheld — anchor status: not_found]
- [methods] specialAas default = KR for decoy generation, and fastaDigestOptions contains the in-silico digestion settings for the provided FASTA.: "specialAas                 | Special amino acids for decoy generation; default = "KR""
- [methods] The job type must be set to SpectralLibraryGeneration among the three supported job types.: "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [abstract] Oktoberfest is an open-source Python package for spectral library generation and rescoring within the Prosit ecosystem.: "we present Oktoberfest, an open source Python package of our spectral library generation and rescoring pipeline"
- [abstract] Prosit deep-learning predictions enable high-quality in silico reference libraries.: [evidence span withheld — anchor status: not_found]
- [methods] Koina server interface provides access to prediction models for the library pipeline.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
