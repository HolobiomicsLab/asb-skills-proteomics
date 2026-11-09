---
name: psm-selection-by-search-score
description: 'Use when building a Prosit-based collision energy calibration set: search
  results (raw/mzml/d/hdf workflows) must be filtered to the highest-confidence PSMs
  before querying Koina for fragment-intensity predictions across an NCE range.'
when_to_use_negative:
- The job type is 'SpectralLibraryGeneration' or 'Rescoring' rather than 'CollisionEnergCalibration'
  — those pipelines use different peptide filtering steps (pp.filter_peptides, pp.filter_peptides_for_model)
- You need de novo or unassigned spectra — this skill relies on scored PSMs from a
  completed database search, so unsearched or unscored spectra cannot be selected
- Only a single global NCE is needed across all files — this method calibrates individually
  per MS file and will not produce one shared NCE
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Oktoberfest
  role: Python package that implements the CollisionEnergCalibration job, including
    PSM filtering by search score, per-file NCE calibration, and output of optimal
    NCE values
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: spectrum_io
  role: Converts raw/d inputs to mzml/hdf and loads spectra so PSMs can be matched
    to observed fragment peaks
  repo: https://github.com/wilhelm-lab/spectrum_io
- name: Koina
  role: Prediction server queried through Oktoberfest's pr.Predictor interface to
    obtain Prosit fragment-intensity predictions for each selected PSM at every candidate
    NCE
- name: Prosit
  role: Deep-learning model served via Koina whose predicted spectra are compared
    to observed spectra (mean spectral angle) to select the optimal NCE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:e8ae9b4397d137cc9396983cd31a67f4b2dd2f81e1142855b7ea333ee9b560f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Prosit
  - Python
  - spectrum_io
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# psm-selection-by-search-score

## Summary

Selects the top 1000 highest-scoring PSMs per MS file from search engine results as the training basis for Oktoberfest's per-file normalized collision energy (NCE) calibration with Prosit predictions.

## When to use

Use when building a Prosit-based collision energy calibration set: search results (raw/mzml/d/hdf workflows) must be filtered to the highest-confidence PSMs before querying Koina for fragment-intensity predictions across an NCE range.

## When NOT to use

- The job type is 'SpectralLibraryGeneration' or 'Rescoring' rather than 'CollisionEnergCalibration' — those pipelines use different peptide filtering steps (pp.filter_peptides, pp.filter_peptides_for_model)
- You need de novo or unassigned spectra — this skill relies on scored PSMs from a completed database search, so unsearched or unscored spectra cannot be selected
- Only a single global NCE is needed across all files — this method calibrates individually per MS file and will not produce one shared NCE

## Inputs

- Search engine results (search output convertible via pp.convert_search / pp.load_search / pp.split_search)
- Mass spectrometry raw files in raw, mzml, or d format (or pre-converted hdf)
- Job configuration with type 'CollisionEnergCalibration' including ce_alignment_options.ce_range = (18,50)
- Mass tolerance settings (default 20 FTMS, 40 TOF, 0.35 ITMS; ppm for FTMS/TOF, da for ITMS)

## Outputs

- Filtered set of top 1000 highest-scoring PSMs per MS file
- Per-file optimal NCE values written to the configured output folder
- Mean spectral angle per candidate NCE per file

## How to apply

After loading search results and spectra for each MS file (formats raw/mzml/d/hdf via spectrum_io conversion utilities such as pp.convert_raw_to_mzml, pp.convert_d_to_hdf, and pp.load_spectra), rank PSMs by their search-engine score and retain only the top 1000 highest-scoring PSMs per file. These PSMs are then paired with Prosit fragment-intensity predictions retrieved from the Koina server (default 'koina.wilhelmlab.org:443', ssl=true) at every candidate NCE in the ce_range (18,50), which is end-exclusive so NCE 18–49 are tested. The rationale is that only confidently identified peptides provide reliable observed spectra for computing the mean spectral angle used to select the optimal NCE. Filtering is performed per MS file so each file receives its own calibration; an optional RANSAC regression refinement (use_ransac_model=true) is recommended for timsTOF data.

## Related tools

- **Oktoberfest** (Python package that implements the CollisionEnergCalibration job, including PSM filtering by search score, per-file NCE calibration, and output of optimal NCE values) — https://github.com/wilhelm-lab/oktoberfest
- **spectrum_io** (Converts raw/d inputs to mzml/hdf and loads spectra so PSMs can be matched to observed fragment peaks) — https://github.com/wilhelm-lab/spectrum_io
- **Koina** (Prediction server queried through Oktoberfest's pr.Predictor interface to obtain Prosit fragment-intensity predictions for each selected PSM at every candidate NCE)
- **Prosit** (Deep-learning model served via Koina whose predicted spectra are compared to observed spectra (mean spectral angle) to select the optimal NCE)

## Examples

```
import oktoberfest as ok; ok.pp.convert_raw_to_mzml(...); ok.pp.load_spectra(...); predictor = ok.pr.Predictor(server='koina.wilhelm-lab.org:443', ssl=True)  # then calibrate NCE over ce_range (18,50) per MS file
```

## Evaluation signals

- Exactly the top 1000 highest-scoring PSMs are retained per MS file, not fewer and not per-run global pooling
- Each MS file yields its own optimal NCE value written to the configured output folder
- Calibration reference: the C*12:03 file yields NCE 32 with mean spectral angle ~0.86 as reported in Figure 2A
- All tested NCE values fall within the configured ce_range (18,50), i.e. integers 18–49 inclusive (end-exclusive upper bound)
- Predicted/observed peak annotation respects the configured massTolerance (20 FTMS, 40 TOF, 0.35 ITMS) and unitMassTolerance (ppm for FTMS/TOF, da for ITMS)

## Limitations

- NCE calibration is performed individually per MS file; results do not transfer automatically between files or instruments
- The ce_range (18,50) is end-exclusive, so NCE 50 itself is never tested
- The RANSAC regression refinement (use_ransac_model=true) is recommended specifically for timsTOF data; without it, calibration on timsTOF may be less reliable
- Selection depends on search-engine score quality — poorly searched files will pass low-confidence PSMs into the calibration set

## Evidence

- [results] NCE calibration in Oktoberfest tests NCE values spanning 18 to 49 and is performed individually per MS file: [evidence span withheld — anchor status: not_found]
- [readme] Oktoberfest is a python tool for collision energy calibration, rescoring search results and generating spectral libraries for proteomics research: [evidence span withheld — anchor status: external_doc]
- [methods] The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring": "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [methods] Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [readme] It provides the following functionalities: calculation of theoretical peptide / ion masses, annotation of spectra, spectral similarity: "calculation of theoretical peptide / ion masses
-   annotation of spectra
-   spectral similarity"
