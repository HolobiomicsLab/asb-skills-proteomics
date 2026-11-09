---
name: precursor-interference-qc-score-calculation
description: Use when quantifying iTRAQ, TMT (up to 11-plex), or TMTpro (up to 16-plex)
  labeled peptides from mzML files plus PSM hit files and you need QC scores (ms1PrecSignal,
  peptideIntensityScore, msnTotalSignal) to filter compression.
when_to_use_negative:
- Input is label-free or uses non-isobaric quantification (the scores target co-isolation
  interference of iTRAQ/TMT/TMTpro reporter ions only)
- The PISA-style RTS-MS3 acquisition acquired only a maximum of 2 PSMs per protein,
  which severely limits filtering possibilities
- Data are already filtered peptide quantification tables — this skill operates on
  raw mzML scans plus PSM hits prior to protein-level quantification
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java-based standalone tool that computes the QC scores, quantifies reporter
    ions, and writes the isofrag/isolab/isopep/isopep_extra CSVs
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: ProteoWizard tool used to convert instrument raw files to mzML (via Docker
    image chambm/pwiz-skyline-i-agree-to-the-vendor-licenses) before scoring
- name: Comet
  role: Search engine whose PSM output is converted into the comma-separated hits
    file consumed by IsobaricQuant
- name: Java 17
  role: Runtime dependency required to execute the IsobaricQuant jar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:669cbcf898f957c8acafe04ea42848a96ebb7c758576a756f98708705c1416e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Java 17
  - msconvert
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# precursor-interference-qc-score-calculation

## Summary

Compute IsobaricQuant QC scores that quantify precursor and reporter ion interference in isobarically-labeled (TMT/TMTpro/iTRAQ) peptide quantification, so scans with ratio compression can be filtered using MS1 precursor purity, MS2 peptide purity, and MS3 SPS purity metrics.

## When to use

Use when quantifying iTRAQ, TMT (up to 11-plex), or TMTpro (up to 16-plex) labeled peptides from mzML files plus PSM hit files and you need QC scores (ms1PrecSignal, peptideIntensityScore, msnTotalSignal) to filter compression.

## When NOT to use

- Input is label-free or uses non-isobaric quantification (the scores target co-isolation interference of iTRAQ/TMT/TMTpro reporter ions only)
- The PISA-style RTS-MS3 acquisition acquired only a maximum of 2 PSMs per protein, which severely limits filtering possibilities
- Data are already filtered peptide quantification tables — this skill operates on raw mzML scans plus PSM hits prior to protein-level quantification

## Inputs

- mzML file of the MS run (converted from raw via msconvert with peakPicking true 1- and zeroSamples removeExtra filters)
- CSV file containing peptide spectral matches (PSM) from a search engine such as Comet (columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- User-supplied text configuration file (scanLevel, PPMTolerance 20, MS1PPMTolerance 20, MS2PPMTolerance 20, topXNum 10)
- Optional reporter ion isotopic distribution CSV for isotopic impurity correction

## Outputs

- isofrag.csv (fragment ion assignments per scan)
- isolab.csv (matched isobaric label intensities, non-negative, within ppm tolerance)
- isopep.csv (per-PSM quantification)
- isopep_extra.csv (per-PSM QC scores: ms1PrecSignal, msnTotalSignal/msnTotalScore, peptideIntensityScore, peptideTopXIntensityFromTotalScore)

## How to apply

Run IsobaricQuant (Java 17) with a user-supplied text config (scanLevel, PPMTolerance 20, MS1PPMTolerance 20, MS2PPMTolerance 20, topXNum 10) against the mzML file and a comma-separated PSM hits file. Compute scan noise as the average intensity of peaks outside the label m/z windows, excluding MS1 non-fragmented precursor M+H, TMT reporter ions, and TMTpro/TMT complementary ion clusters from MS2 score calculations. ms1PrecSignal is the precursor signal percentage (precIntensity/allIntensities) within the ms1PrecWindowDaltons (0.35 Da) window of the MS1 scan; msnTotalSignal sums found isobaric label intensities by nearest-peak search within +/- PPMTolerance; peptideIntensityScore and peptideTopXIntensityFromTotalScore match b/y fragment ions (with neutral losses 63.98, 18.01, 17.03) to scan peaks using topX = 10 to equal the number of SPS-MS3 selection notches. Collect the four output CSVs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) and filter with dataset-optimized cutoffs, using the 25th percentile as a starting point (e.g. msnTotalSignal 1e6 for MS2; peptideIntensityScore 0.25 and peptideTopXIntensityFromTotalScore 0.15 for MS3).

## Related tools

- **IsobaricQuant** (Java-based standalone tool that computes the QC scores, quantifies reporter ions, and writes the isofrag/isolab/isopep/isopep_extra CSVs) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (ProteoWizard tool used to convert instrument raw files to mzML (via Docker image chambm/pwiz-skyline-i-agree-to-the-vendor-licenses) before scoring)
- **Comet** (Search engine whose PSM output is converted into the comma-separated hits file consumed by IsobaricQuant)
- **Java 17** (Runtime dependency required to execute the IsobaricQuant jar)

## Examples

```
java -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>
```

## Evaluation signals

- Each score column is present in isopep_extra.csv (ms1PrecSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore, msnTotalSignal)
- Score values fall in valid ranges: peptideIntensityScore and peptideTopXIntensityFromTotalScore in [0,1]; ms1PrecSignal as a percentage up to 100%
- Label intensities in isolab.csv are non-negative and every matched peak was found within the configured +/- PPMTolerance (20 ppm)
- MS2 score calculations exclude MS1 non-fragmented precursor M+H, TMT reporter ions, and TMTpro complementary ion cluster peaks as specified
- Filtering with recommended cutoffs (msnTotalSignal 1e6 for MS2; peptideIntensityScore 0.25 and peptideTopXIntensityFromTotalScore 0.15 for MS3) decreases protein CVs without substantially reducing the number of observed proteins

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a good starting value for cutoff optimization
- ms1PrecSignal is a limited predictor of reporter ion interference — a majority of scans show 100% even when quantitative accuracy is reduced
- Stringent filtering improves accuracy and precision but significantly decreases peptide observations, exacerbating missing values across multiple labeled sample sets
- The identity of co-isolated peptides and their reporter ion release efficiency is unknown, preventing ratio decompression
- At the protein level filtering preserves observed protein counts, but at the peptide level the loss of observations is substantial

## Evidence

- [methods] Excluded peaks in MS2 scoring: [evidence span withheld — anchor status: not_found]
- [readme] Java 17 dependency: [evidence span withheld — anchor status: external_doc]
