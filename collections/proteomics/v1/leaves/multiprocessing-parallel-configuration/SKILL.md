---
name: multiprocessing-parallel-configuration
description: Use when benchmarking Oktoberfest rescoring runtime across raw-file combinations
  of varying PSM counts, or when deciding whether enabling multiprocessing for prediction/rescoring
  is worth it.
when_to_use_negative:
- The goal is identification sensitivity or FDR improvement rather than runtime measurement
  — this skill only addresses wall-clock scaling, not rescoring performance gains.
- The data are DIA or spectral library generation jobs — the timing workflow targets
  data-driven rescoring of search engine results; job types 'CollisionEnergCalibration'
  and 'SpectralLibraryGeneration' follow different pipelines.
- Only a single small RAW file is processed — the ~4-fold multiprocessing speedup
  (~1 h vs ~15 min) was reported for 15 MS files and will not be reproducible on one
  file.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Python package whose rescoring pipeline (config with type='Rescoring', pp.*
    conversion and re.* rescoring functions) is executed under single-process and
    multiprocessing settings while timing each step
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Koina
  role: Prediction server (default koina.wilhelmlab.org:443) queried during re.generate_features;
    its prediction step is one of the timed components affected by parallelization
- name: mokapot
  role: Performs target-decoy FDR estimation and rescoring via re.rescore_with_mokapot;
    the final timed step of each run
- name: Prosit
  role: Deep-learning model serving in-silico spectral predictions used to generate
    rescoring features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:0accc6ec530e695f068303da34df74d0701d652a4379d6a2e466961a0cef0e9a
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
yes_rate: 0.5
n_claims: 10
---

# multiprocessing-parallel-configuration

## Summary

Configure Oktoberfest rescoring runs to quantify and exploit multiprocessing speedup: run each HLA raw-file combination twice (numThreads = 1 vs multiprocessing enabled), record per-step and total wall-clock timings, and verify the reported ~4-fold speedup on multi-file workloads.

## When to use

Use when benchmarking Oktoberfest rescoring runtime across raw-file combinations of varying PSM counts, or when deciding whether enabling multiprocessing for prediction/rescoring is worth it.

## When NOT to use

- The goal is identification sensitivity or FDR improvement rather than runtime measurement — this skill only addresses wall-clock scaling, not rescoring performance gains.
- The data are DIA or spectral library generation jobs — the timing workflow targets data-driven rescoring of search engine results; job types 'CollisionEnergCalibration' and 'SpectralLibraryGeneration' follow different pipelines.
- Only a single small RAW file is processed — the ~4-fold multiprocessing speedup (~1 h vs ~15 min) was reported for 15 MS files and will not be reproducible on one file.

## Inputs

- Thermo RAW spectra files (HLA immunopeptidomics, six raw-file combinations of increasing PSM counts)
- Search engine results (mzIdentML) from HLA searches
- Oktoberfest configuration file with type='Rescoring' and spectra_type='raw'
- Prediction server endpoint (koina.wilhelmlab.org:443)

## Outputs

- rescoring_runtimes.csv with per-step (conversion, prediction, rescoring) and total wall-clock timings per run
- Rescored PSM tables with mokapot FDR estimation
- Linear-scaling assessment of runtime vs number of PSMs (Figure 4 benchmark)

## How to apply

Write a configuration file with type set to 'Rescoring', specifying the HLA search results and spectra paths (spectra_type 'raw') in the inputs block. Convert Thermo RAW spectra via pp.convert_raw_to_mzml (and pp.convert_d_to_hdf where applicable), then convert/load search results with pp.convert_search and pp.load_search and generate rescoring features with re.generate_features against the prediction server (default koina.wilhelmlab.org:443) using default fragment mass tolerances (20 FTMS / 40 TOF / 0.35 ITMS). Run target-decoy FDR estimation and rescoring with re.rescore_with_mokapot. Execute the full pipeline for each of the six HLA raw-file combinations twice — first with numThreads = 1 (single process), then with multiprocessing enabled — recording elapsed time for conversion, prediction, rescoring, and total wall-clock duration of every run. Rationale: the article's Figure 4 shows rescoring runtime scales linearly with PSM count, and multiprocess execution yields a ~4-fold speedup (~1 h single-process vs ~15 min multi-process for 15 MS files), so the timings can be validated against those baselines. Compile timings into rescoring_runtimes.csv and check that conversion accounts for roughly 40% of total time.

## Related tools

- **Oktoberfest** (Python package whose rescoring pipeline (config with type='Rescoring', pp.* conversion and re.* rescoring functions) is executed under single-process and multiprocessing settings while timing each step) — https://github.com/wilhelm-lab/oktoberfest
- **Koina** (Prediction server (default koina.wilhelmlab.org:443) queried during re.generate_features; its prediction step is one of the timed components affected by parallelization)
- **mokapot** (Performs target-decoy FDR estimation and rescoring via re.rescore_with_mokapot; the final timed step of each run)
- **Prosit** (Deep-learning model serving in-silico spectral predictions used to generate rescoring features)

## Examples

```
import oktoberfest as ok; ok.pp.convert_raw_to_mzml(raw_path, out_path)  # run ok.pp/ok.re rescoring steps with numThreads=1, then with multiprocessing, timing each step
```

## Evaluation signals

- Runtime vs number of PSMs is approximately linear across all six HLA raw-file combinations, matching Figure 4 of the paper.
- File conversion accounts for roughly 40% of total run time.
- Multiprocess execution yields approximately a 4-fold speedup (~1 h single-process vs ~15 min multi-process for 15 MS files).
- Per-step timings for conversion, prediction (Koina), and rescoring (mokapot) are all recorded separately in rescoring_runtimes.csv, not just totals.
- Both execution modes (numThreads = 1 and multiprocessing enabled) were run on identical inputs, so timings are directly comparable.

## Limitations

- Reported speedup (~4-fold) is specific to the multi-file HLA benchmark (15 MS files); smaller workloads or different thread counts will not necessarily match it.
- Conversion (RAW to mzML/HDF) dominates runtime (~40%), so parallelizing only prediction/rescoring leaves most of the wall-clock time unaddressed.
- Timings depend on the external prediction server (default koina.wilhelmlab.org:443) and network conditions, which can confound scaling comparisons across runs.
- The article's changelog is empty and no discussion-section runtime caveats are available; validation rests on the Figure 4 linear-scaling claim.

## Evidence

- [other] Figure 4 linear scaling: [evidence span withheld — anchor status: not_found]
- [other] timed dual-mode execution: [evidence span withheld — anchor status: not_found]
- [methods] job type configuration: "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [methods] Koina prediction interface: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [readme] rescoring pipeline scope: [evidence span withheld — anchor status: external_doc]
