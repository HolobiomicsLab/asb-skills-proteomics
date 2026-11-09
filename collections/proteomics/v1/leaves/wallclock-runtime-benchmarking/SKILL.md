---
name: wallclock-runtime-benchmarking
description: 'Use when reproducing the Figure 4 runtime scaling evaluation: six HLA
  search-result datasets of varying PSM counts need per-step (conversion, prediction,
  rescoring) and total wall-clock timings to test linear scaling with PSM count.'
when_to_use_negative:
- The goal is identification performance (FDR, gain in identifications) rather than
  computational cost — this skill times steps only and does not evaluate rescoring
  sensitivity.
- The input is not search-engine PSM results plus RAW spectra (e.g. already-rescored
  mzIdentML or a spectral-library job type such as 'SpectralLibraryGeneration' or
  'CollisionEnergCalibration').
- Only a single small dataset is available — linear scaling and the ~4x speedup claim
  require multiple PSM-count conditions to be verifiable.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Pipeline whose pp.* and re.* functions (pp.convert_raw_to_mzml, pp.convert_search,
    re.generate_features, re.rescore_with_mokapot) are timed step by step
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443) queried by re.generate_features;
    its latency contributes to the timed prediction step
- name: mokapot
  role: Performs target-decoy FDR estimation and rescoring in re.rescore_with_mokapot,
    the timed rescoring step
- name: Prosit
  role: Underlying prediction model served via Koina whose inference time is part
    of the measured rescoring runtime
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:1684f7e19aadf33b7a3a945958939f21c6d0f6f80a13b42441477d7c90986219
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# wallclock-runtime-benchmarking

## Summary

Times each step and the total wall-clock duration of Oktoberfest's data-driven rescoring pipeline across HLA raw-file combinations of increasing PSM counts, to characterize how runtime scales with input size (linear scaling in Figure 4) and quantify the multi-processing speedup.

## When to use

Use when reproducing the Figure 4 runtime scaling evaluation: six HLA search-result datasets of varying PSM counts need per-step (conversion, prediction, rescoring) and total wall-clock timings to test linear scaling with PSM count.

## When NOT to use

- The goal is identification performance (FDR, gain in identifications) rather than computational cost — this skill times steps only and does not evaluate rescoring sensitivity.
- The input is not search-engine PSM results plus RAW spectra (e.g. already-rescored mzIdentML or a spectral-library job type such as 'SpectralLibraryGeneration' or 'CollisionEnergCalibration').
- Only a single small dataset is available — linear scaling and the ~4x speedup claim require multiple PSM-count conditions to be verifiable.

## Inputs

- Thermo RAW spectra files (spectra_type='raw') for six HLA raw-file combinations of increasing PSM counts
- Search engine PSM results for the HLA runs (converted via pp.convert_search / pp.load_search)
- Oktoberfest Rescoring-type configuration file with inputs block specifying search results and spectra paths
- Default prediction server endpoint koina.wilhelmlab.org:443

## Outputs

- rescoring_runtimes.csv with per-step (conversion, prediction, rescoring) and total wall-clock timings for all six runs, single-process and multi-process
- Scaling assessment: linear fit of runtime vs PSM count, ~40% conversion-time share, ~4x multiprocessing speedup comparison

## How to apply

Write a config with type 'Rescoring', pointing to the HLA search results and RAW spectra (spectra_type='raw'). Time the conversion step via ok.pp.convert_raw_to_mzml (and pp.convert_d_to_hdf where applicable), then pp.convert_search/pp.load_search and re.generate_features against the prediction server koina.wilhelmlab.org:443 with default fragment mass tolerances (20 FTMS / 40 TOF / 0.35 ITMS), and finally target-decoy FDR estimation and rescoring with re.rescore_with_mokapot, timing prediction and rescoring. Run the full pipeline for all six HLA raw-file combinations twice — once with numThreads=1 and once with multiprocessing — recording each step and the total duration. Compile timings into rescoring_runtimes.csv and validate three criteria: runtime scales approximately linearly with PSM count, file conversion accounts for roughly 40% of total time, and multi-process execution gives the reported ~4-fold speedup (~1 h single-process vs ~15 min multi-process for 15 MS files).

## Related tools

- **Oktoberfest** (Pipeline whose pp.* and re.* functions (pp.convert_raw_to_mzml, pp.convert_search, re.generate_features, re.rescore_with_mokapot) are timed step by step) — https://github.com/wilhelm-lab/oktoberfest
- **Koina** (Prediction server (koina.wilhelmlab.org:443) queried by re.generate_features; its latency contributes to the timed prediction step)
- **mokapot** (Performs target-decoy FDR estimation and rescoring in re.rescore_with_mokapot, the timed rescoring step)
- **Prosit** (Underlying prediction model served via Koina whose inference time is part of the measured rescoring runtime)

## Examples

```
import oktoberfest as ok, time; t0=time.time(); ok.re.generate_features(job, server='koina.wilhelmlab.org:443'); ok.re.rescore_with_mokapot(job); print(time.time()-t0)
```

## Evaluation signals

- rescoring_runtimes.csv exists with per-step and total times for all six HLA runs in both numThreads=1 and multiprocessing modes
- A plot of total runtime vs PSM count is approximately linear across the six combinations
- File conversion time is roughly 40% of total wall-clock time per run
- Multi-process run shows the reported ~4-fold speedup (~1 h single-process vs ~15 min multi-process for 15 MS files)
- Fragment mass tolerance settings match defaults: 20 FTMS / 40 TOF / 0.35 ITMS, and the prediction server is koina.wilhelmlab.org:443

## Limitations

- Timing depends on the external prediction server (koina.wilhelmlab.org:443); network latency and server load are not controlled by the local run.
- Thermo RAW conversion (pp.convert_raw_to_mzml, pp.convert_d_to_hdf) dominates runtime (~40%), so timing conclusions are sensitive to file-conversion performance rather than rescoring alone.
- The ~4x multiprocessing speedup was reported for 15 MS files; different file counts or hardware may deviate from this figure.

## Evidence

- [methods] Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot": "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [methods] job type options: "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [methods] peak annotation tolerance: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [readme] README pipeline description: [evidence span withheld — anchor status: external_doc]
