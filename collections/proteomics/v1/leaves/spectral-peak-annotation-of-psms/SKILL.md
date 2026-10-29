---
name: spectral-peak-annotation-of-psms
description: Use when MaxQuant (or another search engine) PSM results plus raw/mzML
  spectra must be annotated with predicted fragment intensities to build features
  for data-driven rescoring with mokapot or percolator.
when_to_use_negative:
- Input is already a rescoring feature table or mokapot/percolator output — annotation
  has already been performed
- The goal is spectral library generation or collision energy calibration rather than
  PSM-level rescoring feature generation
- Spectra have no accompanying search engine PSM results to annotate (no peptide assignments
  exist)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Oktoberfest
  role: Provides re.generate_features and the pp.* preprocessing functions that convert
    search results and spectra and perform the peak annotation
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: Prosit
  role: Deep learning model supplying theoretical fragment-intensity predictions against
    which observed peaks are annotated
- name: Koina
  role: Prediction server (koina.wilhelmlab.org:443, ssl) interfaced by pr.Predictor
    to retrieve Prosit fragment-intensity and iRT predictions
- name: spectrum_fundamentals
  role: Underlying package for theoretical peptide/ion mass calculation, spectrum
    annotation, and spectral similarity used within the Oktoberfest rescoring pipeline
  repo: https://github.com/wilhelm-lab/oktoberfest
- name: mokapot
  role: Consumer of the annotated feature tables for PSM-level target/decoy FDR estimation
    (re.rescore_with_mokapot)
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202300112
    title: 'Oktoberfest: Open‐source spectral library generation and rescoring pipeline
      based on Prosit'
  source_content_hash: sha256:8b309d5f2d92fdd451de300c7c1d0ab5e044f7bae59d0ad844df25ab700db5f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wilhelm-lab/oktoberfest
  tools:
  - Koina
  - Oktoberfest
  - Percolator
  - Prosit
  - Python
  - mokapot
  - spectrum_fundamentals
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# spectral-peak-annotation-of-psms

## Summary

Annotate experimental fragment peaks in MS2 spectra of identified PSMs against theoretical fragment masses (b/y ions) derived from Prosit-style peptide property predictions, as implemented by Oktoberfest's re.generate_features step of the rescoring pipeline.

## When to use

Use when MaxQuant (or another search engine) PSM results plus raw/mzML spectra must be annotated with predicted fragment intensities to build features for data-driven rescoring with mokapot or percolator.

## When NOT to use

- Input is already a rescoring feature table or mokapot/percolator output — annotation has already been performed
- The goal is spectral library generation or collision energy calibration rather than PSM-level rescoring feature generation
- Spectra have no accompanying search engine PSM results to annotate (no peptide assignments exist)

## Inputs

- MaxQuant search engine results (e.g., msms.txt / out files from PXD010871 metaproteomics or four-allele HLA immunopeptidomics runs)
- Raw spectra files (Thermo .raw converted to mzML, then HDF5 via pp.convert_d_to_hdf)
- Prosit fragment-intensity and iRT predictions retrieved from a Koina server

## Outputs

- Annotated spectra with experimentally observed fragment peaks matched to theoretical b/y ions
- Rescoring feature tables (spectral angle, intensity-based features) merged with the native search engine score
- Inputs to mokapot/percolator rescoring at 1% PSM-level FDR

## How to apply

Convert the search engine output to Oktoberfest's internal format with pp.convert_search and pp.split_search, convert raw files to mzML (pp.convert_raw_to_mzml, pp.convert_d_to_hdf, pp.load_spectra), and retrieve Prosit fragment-intensity and iRT predictions via pr.Predictor interfacing a Koina server (koina.wilhelmlab.org:443, ssl=true). Then run re.generate_features to annotate observed peaks against theoretical fragment masses, using a massTolerance of 20 for FTMS with unitMassTolerance 'ppm', 40 for TOF, or 0.35 for ITMS (unit selected by mass analyzer), and a precursor peak removal window of p_window 1.2 Da. The annotated spectra plus predicted intensities yield rescoring features (including spectral angle and intensity-based features) that are merged with the native search score (re.merge_input) before FDR-controlled rescoring at 1% PSM-level target/decoy FDR. Correct tolerance selection by analyzer type is the key decision point, since the tolerance defines what counts as a match between theoretical and experimentally observed fragment mass.

## Related tools

- **Oktoberfest** (Provides re.generate_features and the pp.* preprocessing functions that convert search results and spectra and perform the peak annotation) — https://github.com/wilhelm-lab/oktoberfest
- **Prosit** (Deep learning model supplying theoretical fragment-intensity predictions against which observed peaks are annotated)
- **Koina** (Prediction server (koina.wilhelmlab.org:443, ssl) interfaced by pr.Predictor to retrieve Prosit fragment-intensity and iRT predictions)
- **spectrum_fundamentals** (Underlying package for theoretical peptide/ion mass calculation, spectrum annotation, and spectral similarity used within the Oktoberfest rescoring pipeline) — https://github.com/wilhelm-lab/oktoberfest
- **mokapot** (Consumer of the annotated feature tables for PSM-level target/decoy FDR estimation (re.rescore_with_mokapot))

## Examples

```
import oktoberfest as ok  # ok.re.generate_features(...) with massTolerance=20, unitMassTolerance='ppm' for FTMS and p_window=1.2
```

## Evaluation signals

- Mass tolerance settings match the mass analyzer: 20 ppm for FTMS, 40 for TOF, 0.35 for ITMS, with unitMassTolerance (da/ppm) chosen by analyzer
- Precursor peak removal window p_window is set to 1.2 Da so precursor peaks do not contaminate fragment annotations
- Downstream rescoring reproduces the reported gains, e.g., metaproteomics (Swissprot+IGC) rescoring yields 114,510 PSMs vs 434 for MaxQuant-only, and HLA rescoring reaches 95,451 PSMs vs 78,842 previously
- Annotated features include spectral angle and intensity-based features and merge correctly with the native Andromeda/MaxQuant score via re.merge_input
- Feature tables pass 1% PSM-level target/decoy FDR estimation with the chosen FDR estimation method ('mokapot' or 'percolator')

## Limitations

- Tolerance must be matched to the instrument's mass analyzer; applying ppm tolerances to ion-trap (ITMS) data or vice versa will mis-annotate peaks
- Peak annotation requires Prosit predictions via a Koina server, so a network connection to koina.wilhelmlab.org:443 (ssl) is needed
- Results depend on the upstream search engine format being correctly converted (pp.convert_search/pp.split_search) before annotation

## Evidence

- [methods] Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation: "Defines the allowed tolerance between theoretical and experimentally observered fragment mass during peak annotation"
- [methods] Koina prediction interface: "Access to functions that interface either a Koina server to retrieve predictions from various prediction models."
- [methods] FDR estimation method choice: "Method used for target / decoy separation and FDR estimation on PSM and peptide level: "percolator" or "mokapot""
- [readme] Oktoberfest scope from README: [evidence span withheld — anchor status: external_doc]
