---
name: collision-energy-calibration
description: Use when search-engine results and spectra (raw/mzml/d/hdf) are available
  and you need the per-file optimal NCE for Prosit/Koina spectral predictions, e.g.
  before spectral library generation or rescoring.
when_to_use_negative:
- You only need to rescore search results with mokapot/percolator — run the 'Rescoring'
  job type instead of calibration
- You already have instrument-reported collision energies and only need spectral library
  generation ('SpectralLibraryGeneration' job type)
- Input spectra lack search results, so no high-scoring PSMs can be selected to anchor
  the calibration
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3522
- http://edamontology.org/topic_0092
tools:
- name: Oktoberfest
  role: 'Runs the CollisionEnergCalibration job: PSM selection, peak annotation, spectral-angle
    computation and NCE selection per file'
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model providing fragment-intensity predictions at each candidate
    NCE, retrieved via the Koina interface
- name: Koina
  role: Prediction server (default 'koina.wilhelmlab.org:443', ssl=true) queried by
    Oktoberfest's pr.Predictor to obtain Prosit predictions
- name: spectrum_io
  role: Loads/converts the input MS files (raw/mzml/d/hdf) for the calibration workflow
  repo: wilhelm-lab/spectrum_io
- name: Python
  role: Runtime language for Oktoberfest (Python 3.10+), e.g. 'import oktoberfest
    as ok'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:0eede9539ba64f0061511e5d86cb3de697e7d0bd3b80dd2fc239ba4e77ea8b42
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
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3522
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# collision-energy-calibration

## Summary

Determine the optimal normalized collision energy (NCE) per MS file by comparing Prosit-predicted fragment intensities across a candidate NCE range against observed spectra, as implemented in Oktoberfest's CollisionEnergCalibration job type.

## When to use

Use when search-engine results and spectra (raw/mzml/d/hdf) are available and you need the per-file optimal NCE for Prosit/Koina spectral predictions, e.g. before spectral library generation or rescoring.

## When NOT to use

- You only need to rescore search results with mokapot/percolator — run the 'Rescoring' job type instead of calibration
- You already have instrument-reported collision energies and only need spectral library generation ('SpectralLibraryGeneration' job type)
- Input spectra lack search results, so no high-scoring PSMs can be selected to anchor the calibration

## Inputs

- Search engine results (PSMs) for each MS file
- Spectra in raw, mzML, or Thermo .d / hdf format
- Job configuration JSON with type 'CollisionEnergCalibration' and ce_alignment_options.ce_range (18,50)
- Access to a Koina prediction server (e.g. koina.wilhelmlab.org:443, ssl=true)

## Outputs

- Per-file optimal NCE values written to the configured output folder
- Mean spectral angle values per candidate NCE (e.g. ~0.86 at NCE 32 for the reference file)
- Optionally a RANSAC regression model refining NCE alignment

## How to apply

Parse the job configuration with type 'CollisionEnergCalibration' and set ce_alignment_options.ce_range to (18,50), which is end-exclusive so NCE 18–49 are tested. Load spectra for each MS file (raw/mzml/d/hdf via spectrum_io) and retain the top 1000 highest-scoring PSMs per file. Query the Koina server (default 'koina.wilhelmlab.org:443', ssl=true) through the pr.Predictor interface to get Prosit fragment-intensity predictions for each PSM at every candidate NCE. Annotate experimental peaks against predictions using massTolerance (default 20 FTMS, 40 TOF, 0.35 ITMS) and unitMassTolerance (ppm for FTMS/TOF, da for ITMS), then select, per file, the NCE maximizing the mean spectral angle between predicted and observed spectra. Optionally refine with the RANSAC regression model (use_ransac_model=true, recommended for timsTOF data) and write the per-file optimal NCE values to the output folder.

## Related tools

- **Oktoberfest** (Runs the CollisionEnergCalibration job: PSM selection, peak annotation, spectral-angle computation and NCE selection per file) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model providing fragment-intensity predictions at each candidate NCE, retrieved via the Koina interface)
- **Koina** (Prediction server (default 'koina.wilhelmlab.org:443', ssl=true) queried by Oktoberfest's pr.Predictor to obtain Prosit predictions)
- **spectrum_io** (Loads/converts the input MS files (raw/mzml/d/hdf) for the calibration workflow) — wilhelm-lab/spectrum_io
- **Python** (Runtime language for Oktoberfest (Python 3.10+), e.g. 'import oktoberfest as ok')

## Examples

```
import oktoberfest as ok
```

## Evaluation signals

- Per-file output lists exactly one optimal NCE selected as the maximum mean spectral angle over NCE 18–49 (range end-exclusive)
- Reference validation: the C*12:03 file yields NCE 32 with mean spectral angle ~0.86 (Figure 2A)
- PSM filtering retains the top 1000 highest-scoring PSMs per MS file before prediction
- Peak annotation respects the configured tolerances: massTolerance 20 FTMS / 40 TOF / 0.35 ITMS, unitMassTolerance ppm (FTMS/TOF) or da (ITMS)
- Optimal NCE typically falls around 30–34 NCE units for common instrument data; large deviations warrant inspection

## Limitations

- NCE calibration is performed individually per MS file, so one global NCE is not inferred across files
- The RANSAC refinement model is optional and specifically recommended for timsTOF data; results without it may be less robust on that platform
- Prediction quality depends on access to a reachable Koina server (koina.wilhelmlab.org:443 over SSL)

## Evidence

- [methods] The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring": "The type of job; can be one of "CollisionEnergCalibration", "SpectralLibraryGeneration" or "Rescoring""
- [methods] Tolerances for peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [methods] Prediction interface: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [readme] Tool scope from README: [evidence span withheld — anchor status: external_doc]
