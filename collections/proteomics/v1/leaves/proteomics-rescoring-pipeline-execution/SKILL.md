---
name: proteomics-rescoring-pipeline-execution
description: Use when raw spectra (Thermo RAW/mzML) plus search engine results must
  be rescored with Prosit-predicted intensities, or when benchmarking Oktoberfest
  rescoring runtime scaling across files of increasing PSM counts.
when_to_use_negative:
- The goal is spectral library generation or collision energy calibration — use the
  'SpectralLibraryGeneration' or 'CollisionEnergCalibration' job types instead of
  'Rescoring'.
- Search results have already been rescored and FDR-estimated with mokapot/percolator
  — no rescoring step is needed.
- No Prosit/Koina prediction server access is available and no local prediction endpoint
  can be configured, since feature generation requires peptide property predictions.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Main Python pipeline providing pp.convert_raw_to_mzml, pp.convert_search,
    re.generate_features and re.rescore_with_mokapot for the rescoring workflow
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep-learning peptide property (fragment intensity) prediction model used
    to create in silico reference features for rescoring
- name: Koina
  role: Prediction server (default koina.wilhelmlab.org:443) interfaced by Oktoberfest
    functions to retrieve Prosit predictions during feature generation
- name: mokapot
  role: Target-decoy separation and FDR estimation on PSM and peptide level during
    the re.rescore_with_mokapot rescoring step
- name: spectrum_io
  role: File/data conversion package (e.g., RAW to mzML/HDF conversion) within the
    Oktoberfest ecosystem
  repo: https://github.com/wilhelm-lab/spectrum_io
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:00bf89082cbe5eb25d5a5a65e4c32b4b9b7cc69cacf70242428f5464899456b0
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
  - spectrum_io
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# proteomics-rescoring-pipeline-execution

## Summary

Executes Oktoberfest's data-driven rescoring pipeline: converts raw spectra and search engine results, generates Prosit/Koina-based rescoring features, and performs target-decoy FDR estimation and rescoring with mokapot. Also supports benchmarking per-step and total runtimes of the pipeline as a function of PSM count.

## When to use

Use when raw spectra (Thermo RAW/mzML) plus search engine results must be rescored with Prosit-predicted intensities, or when benchmarking Oktoberfest rescoring runtime scaling across files of increasing PSM counts.

## When NOT to use

- The goal is spectral library generation or collision energy calibration — use the 'SpectralLibraryGeneration' or 'CollisionEnergCalibration' job types instead of 'Rescoring'.
- Search results have already been rescored and FDR-estimated with mokapot/percolator — no rescoring step is needed.
- No Prosit/Koina prediction server access is available and no local prediction endpoint can be configured, since feature generation requires peptide property predictions.

## Inputs

- Thermo RAW spectra files (converted to mzML via pp.convert_raw_to_mzml, or .hdf via pp.convert_d_to_hdf)
- Search engine results from HLA immunopeptidomics runs (converted/loaded via pp.convert_search and pp.load_search)
- Oktoberfest configuration file with type 'Rescoring' and inputs block specifying search results and spectra paths
- Prediction server endpoint (default koina.wilhelmlab.org:443)

## Outputs

- Rescoring features generated from Prosit/Koina predictions (via re.generate_features)
- Rescored PSMs with target-decoy FDR estimates at PSM and peptide level (via re.rescore_with_mokapot)
- rescoring_runtimes.csv with per-step (conversion, prediction, rescoring) and total wall-clock timings per run
- Summary and quality-control figures/plot summaries

## How to apply

Write a configuration file with type set to 'Rescoring' and specify the search results and spectra paths (spectra_type 'raw') in the inputs block. Convert Thermo RAW spectra with pp.convert_raw_to_mzml (and pp.convert_d_to_hdf where applicable), timing the conversion step, then convert and load search results with pp.convert_search and pp.load_search. Generate rescoring features via re.generate_features against the prediction server (default koina.wilhelmlab.org:443) using the default fragment mass tolerances (20 FTMS / 40 TOF / 0.35 ITMS). Perform target-decoy FDR estimation and rescoring with re.rescore_with_mokapot (percolator is the alternative), timing the prediction and rescoring steps. To reproduce the runtime benchmark, run all six HLA raw-file combinations of increasing PSM counts first with numThreads = 1 and then with multiprocessing, recording each step's elapsed time; validation expects approximately linear runtime scaling with PSM count, file conversion at roughly 40% of total time, and a ~4-fold multi-process speedup (~1 h single-process vs ~15 min for 15 MS files).

## Related tools

- **Oktoberfest** (Main Python pipeline providing pp.convert_raw_to_mzml, pp.convert_search, re.generate_features and re.rescore_with_mokapot for the rescoring workflow) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep-learning peptide property (fragment intensity) prediction model used to create in silico reference features for rescoring)
- **Koina** (Prediction server (default koina.wilhelmlab.org:443) interfaced by Oktoberfest functions to retrieve Prosit predictions during feature generation)
- **mokapot** (Target-decoy separation and FDR estimation on PSM and peptide level during the re.rescore_with_mokapot rescoring step)
- **spectrum_io** (File/data conversion package (e.g., RAW to mzML/HDF conversion) within the Oktoberfest ecosystem) — https://github.com/wilhelm-lab/spectrum_io

## Examples

```
import oktoberfest as ok
```

## Evaluation signals

- Runtime scales approximately linearly with the number of PSMs across the six HLA raw-file combinations (Figure 4 benchmark)
- File conversion accounts for roughly 40% of total pipeline runtime
- Multi-process execution yields the reported ~4-fold speedup (~1 h single-process vs ~15 min multi-process for 15 MS files)
- Per-step elapsed times (conversion, prediction, rescoring) and total wall-clock durations are recorded for every run and compiled into rescoring_runtimes.csv
- FDR estimation is performed on both PSM and peptide level with the chosen method ('mokapot' or 'percolator')

## Limitations

- Runtime is dominated by file conversion (~40% of total time), which limits end-to-end throughput on large RAW inputs
- Default fragment mass tolerances are fixed at 20 FTMS / 40 TOF / 0.35 ITMS; deviating instrument types may require configuration changes
- Feature generation requires access to the Koina prediction server (default koina.wilhelmlab.org:443), introducing network dependence
- Large jobs (e.g., 15 MS files) take ~1 h with numThreads = 1; multiprocessing must be enabled to achieve the reported speedup

## Evidence

- [readme] Oktoberfest is a python tool for collision energy calibration, rescoring search results and generating spectral libraries for proteomics research within the Prosit ecosystem.: [evidence span withheld — anchor status: external_doc]
- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [methods] The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring": "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [methods] pp.convert_raw_to_mzml
   pp.convert_d_to_hdf
   pp.load_spectra: "pp.convert_raw_to_mzml
   pp.convert_d_to_hdf
   pp.load_spectra"
- [methods] Access to functions that interface either a Koina server to retrieve predictions from various prediction models.: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
