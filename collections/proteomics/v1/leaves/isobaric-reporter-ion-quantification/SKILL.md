---
name: isobaric-reporter-ion-quantification
description: Use when quantifying TMT/TMTpro/iTRAQ reporter ions from mzML spectra
  plus PSM hits, or when filtering isobarically-labeled scans suffering precursor
  interference/compression prior to protein-level quantification.
when_to_use_negative:
- The data are label-free or not labeled with iTRAQ/TMT/TMTpro isobaric tags, or the
  plex exceeds supported limits (iTRAQ 8-plex, TMT 11-plex, TMTpro 16-plex).
- You need to identify the co-isolated peptides and their reporter ion release efficiency
  to perform ratio decompression — IsobaricQuant cannot resolve these, it only scores
  interference.
- Input is already a protein-level feature table; filtering must be applied at the
  scan/peptide level directly prior to protein-level quantification.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: IsobaricQuant
  role: Standalone Java tool that computes reporter ion intensities, QC scores, and
    outputs the four CSV files
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: ProteoWizard tool (run via Docker) used to convert instrument raw files to
    the mzML format IsobaricQuant requires
- name: Comet
  role: Search engine whose output is converted into the comma-separated PSM hits
    file consumed by IsobaricQuant
- name: Java 17
  role: Runtime dependency required to execute the IsobaricQuant jar
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:1783697072ab2e6ca8599523561f7f3f3c0b529344d910168a66ab5461d67a2d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Java 17
  - Percolator
  - msconvert
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# isobaric-reporter-ion-quantification

## Summary

IsobaricQuant is a Java-based, cross-platform tool that quantifies iTRAQ (up to 8-plex), TMT (up to 11-plex), and TMTpro (up to 16-plex) reporter ion intensities from mzML files and computes QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) to filter scans with compression from co-isolated precursors.

## When to use

Use when quantifying TMT/TMTpro/iTRAQ reporter ions from mzML spectra plus PSM hits, or when filtering isobarically-labeled scans suffering precursor interference/compression prior to protein-level quantification.

## When NOT to use

- The data are label-free or not labeled with iTRAQ/TMT/TMTpro isobaric tags, or the plex exceeds supported limits (iTRAQ 8-plex, TMT 11-plex, TMTpro 16-plex).
- You need to identify the co-isolated peptides and their reporter ion release efficiency to perform ratio decompression — IsobaricQuant cannot resolve these, it only scores interference.
- Input is already a protein-level feature table; filtering must be applied at the scan/peptide level directly prior to protein-level quantification.

## Inputs

- mzML file of the MS run (converted from raw via msconvert)
- CSV hits file of peptide spectral matches from a search engine (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- User-supplied text configuration file (isoMethod, scanLevel, PPMTolerance, topXNum)
- Optional reporter ion isotopic distribution CSV for impurity correction

## Outputs

- isofrag.csv (fragment ion assignments)
- isolab.csv (isobaric label intensities per scan)
- isopep.csv (peptide-level reporter ion quantification)
- isopep_extra.csv (peptide quantification plus QC score columns such as peptideIntensityScore and peptideTopXIntensityFromTotalScore)

## How to apply

Convert instrument raw files to mzML with msconvert (Docker image chambm/pwiz-skyline-i-agree-to-the-vendor-licenses, using --mzML --simAsSpectra --32 --zlib, peakPicking true 1-, zeroSamples removeExtra), and convert search engine (e.g. Comet) output to the comma-separated hits format (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z). Run IsobaricQuant on Java 17 via CLI with a text config specifying the isoMethod (e.g. TMTpro), PPMTolerance/MS1PPMTolerance/MS2PPMTolerance of 20 ppm, scanLevel per acquisition (MS2 or RTS-MS3), and topXNum 10. The tool computes scan noise as the average intensity of peaks outside label m/z windows, excludes MS1 non-fragmented precursor M+H, TMT reporter ions, and TMTpro complementary ion clusters from MS2 scores, and emits four CSVs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv). For filtering, derive cutoffs from the 25th percentile as a starting value; recommended values are msnTotalSignal 1e6 for MS2 data and peptideIntensityScore 0.25 / peptideTopXIntensityFromTotalScore 0.15 (with topX = 10, matching SPS-MS3 selection notches) for MS3 data, applying filtering directly prior to protein-level quantification so protein inference is not altered.

## Related tools

- **IsobaricQuant** (Standalone Java tool that computes reporter ion intensities, QC scores, and outputs the four CSV files) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (ProteoWizard tool (run via Docker) used to convert instrument raw files to the mzML format IsobaricQuant requires)
- **Comet** (Search engine whose output is converted into the comma-separated PSM hits file consumed by IsobaricQuant)
- **Java 17** (Runtime dependency required to execute the IsobaricQuant jar)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o output
```

## Evaluation signals

- All four output CSVs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) exist and parse correctly, with each documented score column present in isopep_extra.csv.
- Score values fall in valid ranges: e.g. peptideIntensityScore in [0,1] and ms1PrecSignal percentages up to 100%.
- Label intensities in isolab.csv are non-negative and matched peaks are recorded within the configured ppm tolerance (e.g. 20 ppm).
- Cross-tool validation: log10 reporter intensities from IsobaricQuant correlate (Pearson) with FragPipe and MaxQuant on shared scan groups; on the HYpro16 TMTpro benchmark the fraction of shared yeast scan groups with all 16 channels quantified reaches 100% at MS2 level.

## Limitations

- ms1PrecSignal is a limited predictor of reporter ion interference: a majority of scans show 100% even when quantitative accuracy is reduced.
- The optimal QC score and cutoff are dataset-dependent (slightly different score distributions across datasets), and cutoffs are not universally applicable.
- Filtering interfered quantifications improves accuracy and precision but significantly decreases peptide observations, potentially exacerbating missing values across multiple labeled sample sets.
- RTS-MS3 acquisition limited to a maximum of 2 PSMs per protein (as in the PISA experiment) significantly limits filtering possibilities; higher PSMs per protein are recommended.

## Evidence
