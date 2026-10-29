---
name: raw-mass-spectrometry-peak-matching
description: Reference for locating the nearest observed peak to a theoretical m/z
  (reporter ions, precursors, b/y fragments with neutral losses) in mzML scans under
  a ppm tolerance, e.g. to compute QC scores for isobarically-labeled peptide quantification.
when_to_use_negative:
- Input is already a quantified feature/PSM table with reporter intensities — peak
  matching against raw scans is redundant
- 'Data are label-free or use non-isobaric tags: IsobaricQuant only supports iTRAQ
  up to 8-plex, TMT up to 11-plex and TMTpro up to 16-plex'
- The mzML was not peak-picked or retains zero-intensity samples, so nearest-peak
  search within +/- PPMTolerance is unreliable
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java tool that performs the peak matching of isobaric label m/z, precursor,
    and fragment ions against mzML peaks to compute QC scores
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: ProteoWizard tool used to convert instrument raw files to peak-picked mzML
    suitable for peak matching
- name: Comet
  role: Search engine whose PSM output is converted into the comma-separated hits
    file feeding the peak-matching step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:1bd9f2a1419bfd7bdd21e66226f64b78c8969e987ef9403f16efb9ca02e06cd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - msconvert
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Raw mass spectrometry peak matching

## Summary

Matches theoretical m/z values (isobaric label reporter ions, precursor ions, and peptide fragment ions) to observed peaks in mzML scans within a fixed ppm tolerance to compute signal and purity scores used by IsobaricQuant for interference filtering.

## When to use

Reference for locating the nearest observed peak to a theoretical m/z (reporter ions, precursors, b/y fragments with neutral losses) in mzML scans under a ppm tolerance, e.g. to compute QC scores for isobarically-labeled peptide quantification.

## When NOT to use

- Input is already a quantified feature/PSM table with reporter intensities — peak matching against raw scans is redundant
- Data are label-free or use non-isobaric tags: IsobaricQuant only supports iTRAQ up to 8-plex, TMT up to 11-plex and TMTpro up to 16-plex
- The mzML was not peak-picked or retains zero-intensity samples, so nearest-peak search within +/- PPMTolerance is unreliable

## Inputs

- mzML file of the MS run (converted from raw via msconvert with peakPicking and zeroSamples filters)
- CSV file of peptide spectral matches (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) from a search engine such as Comet
- User-supplied text configuration file (scanLevel, PPMTolerance, MS1PPMTolerance, MS2PPMTolerance, topXNum, ms1PrecWindowDaltons, searchMethod)
- Optional reporter ion isotopic distribution CSV for impurity correction

## Outputs

- isofrag.csv (matched fragment ion assignments per scan)
- isolab.csv (per-label reporter ion intensities matched within ppm tolerance)
- isopep.csv (per-PSM quantification)
- isopep_extra.csv (per-PSM QC scores: ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore)

## How to apply

Parse the mzML scan file and the comma-separated PSM hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) with the documented config (scanLevel, PPMTolerance 20, MS1PPMTolerance 20, MS2PPMTolerance 20, topXNum 10). For each isobaric label, search the label's m/z for the nearest observed peak within +/- PPMTolerance using the configured searchMethod, and sum matched intensities as msnTotalSignal. For MS1 precursor purity, compute signalPerc = precIntensity/allIntensities within the ms1PrecWindowDaltons (0.35 Da) window; for scan noise, average intensities of peaks outside the label m/z windows extended by +/- PPMTolerance. When matching fragment ions for MS2 peptide purity, match b/y ions (with neutral losses 63.98, 18.01, 17.03) to scan peaks, excluding MS1 non-fragmented precursor M+H, TMT reporter ions, and TMTpro/TMT complementary ion cluster series, then derive peptideIntensityScore = totalFragmIonsIntensity/totalIntensity and peptideTopXIntensityFromTotalScore with topX = 10. Correct matching is judged by matched peaks falling inside the ppm tolerance, non-negative label intensities, and score values in valid ranges (e.g. peptideIntensityScore in [0,1]).

## Related tools

- **IsobaricQuant** (Java tool that performs the peak matching of isobaric label m/z, precursor, and fragment ions against mzML peaks to compute QC scores) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (ProteoWizard tool used to convert instrument raw files to peak-picked mzML suitable for peak matching)
- **Comet** (Search engine whose PSM output is converted into the comma-separated hits file feeding the peak-matching step)

## Examples

```
java -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>
```

## Evaluation signals

- Every matched label peak in isolab.csv is recorded within the configured +/- PPMTolerance (20 ppm) and intensities are non-negative
- All QC score columns are present in isopep_extra.csv and values fall in valid ranges (peptideIntensityScore in [0,1], ms1PrecSignal percentages up to 100%)
- ms1PrecSignal equals precIntensity/allIntensities computed only within the 0.35 Da ms1PrecWindowDaltons window of the MS1 scan
- Fragment-ion matches are restricted to b/y ions plus neutral losses 63.98, 18.01, 17.03, with excluded peaks (MS1 M+H, TMT reporter ions, TMTpro complementary clusters) removed from MS2 score calculations
- Four output CSVs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) are written with the documented columns

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- ms1PrecSignal is a limited predictor of reporter ion interference — a majority of scans show 100% even when quantitative accuracy is reduced
- Stringent filtering can substantially decrease peptide observations and exacerbate missing values across multiple labeled sample sets
- Java 17 is required to run IsobaricQuant; older Java versions will not work

## Evidence

- [methods] Nearest-peak label search: [evidence span withheld — anchor status: not_found]
- [methods] Excluded peaks in MS2 scoring: [evidence span withheld — anchor status: not_found]
- [readme] Recommended msconvert conversion: [evidence span withheld — anchor status: external_doc]
