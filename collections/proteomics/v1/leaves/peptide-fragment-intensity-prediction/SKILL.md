---
name: peptide-fragment-intensity-prediction
description: Use when search engine PSMs, peptide lists (.csv), or proteome FASTA
  inputs need Prosit fragment-intensity predictions — e.g., to build rescoring features
  with re.generate_features, calibrate NCE, or assemble Spectronaut/.msp spectral
  libraries.
when_to_use_negative:
- Input spectra were acquired on timsTOF without the Prosit-timsTOF model — the default
  Prosit predictions and NCE calibration (RANSAC refinement recommended for timsTOF)
  may not transfer
- The goal is quantification or protein-group FDR rather than identification/rescoring
  — use the dedicated quantification FDR approach cited in the README instead
- Predictions are needed for fragmentation methods outside the configured 'HCD' or
  'CID' options supported by the library-generation settings
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Prosit
  role: Deep learning model providing peptide fragment intensity and iRT predictions
- name: Koina
  role: Prediction server interface (koina.wilhelmlab.org:443) used by pr.Predictor/pr.Koina
    to retrieve Prosit predictions
- name: Oktoberfest
  role: Python package wrapping prediction, peak annotation, rescoring (re.generate_features,
    re.rescore_with_mokapot) and library writing (pp.gen_lib)
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Consumes prediction-derived features for target/decoy separation and PSM/peptide-level
    FDR estimation
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:20e7f9e84ae84810b0a2b9ff76dcd3d189600b5b56a99baa5224592dd1cc4451
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# peptide-fragment-intensity-prediction

## Summary

Predict peptide fragment intensities (and iRT) with the Prosit deep learning model served via Koina, using the Oktoberfest pr.Predictor/Koina interface, to power data-driven rescoring, collision energy calibration, and spectral library generation for proteomics.

## When to use

Use when search engine PSMs, peptide lists (.csv), or proteome FASTA inputs need Prosit fragment-intensity predictions — e.g., to build rescoring features with re.generate_features, calibrate NCE, or assemble Spectronaut/.msp spectral libraries.

## When NOT to use

- Input spectra were acquired on timsTOF without the Prosit-timsTOF model — the default Prosit predictions and NCE calibration (RANSAC refinement recommended for timsTOF) may not transfer
- The goal is quantification or protein-group FDR rather than identification/rescoring — use the dedicated quantification FDR approach cited in the README instead
- Predictions are needed for fragmentation methods outside the configured 'HCD' or 'CID' options supported by the library-generation settings

## Inputs

- Search engine results (e.g., unfiltered MaxQuant/Andromeda output of HLA immunopeptidomics data)
- Spectra files in raw/mzml/d/hdf formats (RAW converted to mzML via ThermoRawFileParser)
- .csv file of requested precursor peptides or .fasta proteome for in-silico spectral library generation
- Prediction configuration: model name, fragmentation method (HCD/CID), collision energy, precursor charges, UniMod modification IDs/masses

## Outputs

- Prosit fragment intensity and iRT predictions per PSM/peptide
- Rescoring feature tables for mokapot/percolator FDR estimation
- Per-file optimal NCE values (e.g., NCE 32, mean spectral angle ~0.86 for the C*12:03 file)
- Spectronaut-compatible .csv and .msp spectral library files

## How to apply

Retrieve Prosit fragment-intensity and iRT predictions from the Koina server (default koina.wilhelmlab.org:443, ssl=true) through the Oktoberfest pr.Predictor/pr.Koina interface. For rescoring, annotate experimental peaks against predicted spectra using a fragment mass tolerance of 20 (FTMS) / 40 (TOF) / 0.35 (ITMS) with unitMassTolerance 'ppm' for FTMS/TOF or 'da' for ITMS, and fragmentation_method 'HCD'; the resulting features feed re.generate_features and re.rescore_with_mokapot. For NCE calibration, query predictions at NCE values spanning 18–49 per MS file, then select the NCE maximizing the mean spectral angle between predicted and observed spectra. For spectral library generation, apply fragmentation 'HCD'/'CID', collision energy 30 (default), precursor charges [2,3], at most 1 methionine oxidation (nrOx = 1), and a minimal relative intensity threshold of 5e-4 before writing libraries in 'spectronaut' and 'msp' formats via pp.gen_lib.

## Related tools

- **Prosit** (Deep learning model providing peptide fragment intensity and iRT predictions)
- **Koina** (Prediction server interface (koina.wilhelmlab.org:443) used by pr.Predictor/pr.Koina to retrieve Prosit predictions)
- **Oktoberfest** (Python package wrapping prediction, peak annotation, rescoring (re.generate_features, re.rescore_with_mokapot) and library writing (pp.gen_lib)) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Consumes prediction-derived features for target/decoy separation and PSM/peptide-level FDR estimation)

## Examples

```
import oktoberfest as ok; from oktoberfest import pr; preds = pr.Predictor(...)
```

## Evaluation signals

- Rescoring gains on the C*12:03 HLA dataset match reported Figure 2C–D: ~2574 uniquely identified peptides (~489% gain) and ~14,318 PSMs (~296% gain), with only 7 peptides / 49 PSMs lost
- NCE calibration on the C*12:03 file yields NCE 32 with mean spectral angle ~0.86 as reported in Figure 2A
- Peak annotation respects analyzer-specific mass tolerances: 20 (FTMS) / 40 (TOF) / 0.35 (ITMS) with 'ppm' or 'da' unitMassTolerance per mass analyzer
- Generated spectral library files are present in the output folder in both requested formats (Spectronaut-compatible .csv and .msp)
- Predictions are retrievable from the Koina server with ssl=true at the default endpoint koina.wilhelmlab.org:443

## Limitations

- NCE must be calibrated individually per MS file; a single global NCE is not assumed optimal
- RANSAC regression refinement of collision energy calibration is recommended for timsTOF data (use_ransac_model=true)
- Discussion-section provenance is incomplete: no changelog or original publication date metadata were available in the extracted source

## Evidence

- [abstract] ML and deep learning (DL) models for peptide property prediction such as Prosit have enabled the creation of high quality in silico reference libraries: [evidence span withheld — anchor status: not_found]
- [methods] Koina server interface: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [methods] peak annotation tolerance definition: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [readme] README pipeline description: [evidence span withheld — anchor status: external_doc]
