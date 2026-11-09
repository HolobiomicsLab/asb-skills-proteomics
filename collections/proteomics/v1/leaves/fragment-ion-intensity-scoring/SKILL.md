---
name: fragment-ion-intensity-scoring
description: Use when iTRAQ/TMT/TMTpro-labeled PSMs need interference QC scores computed
  per scan from b/y fragment ion intensity fractions in mzML data, to filter compressed
  reporter ion quantifications.
when_to_use_negative:
- Input is a protein-level feature table or already-filtered quantification matrix
  — the scores must be computed per scan prior to protein-level quantification.
- Data are label-free or use non-isobaric labeling (the score calculations revolve
  around iTRAQ/TMT/TMTpro reporter and complementary ion series).
- Only MS1 precursor purity assessment is needed — ms1PrecSignal is a limited predictor
  of reporter ion interference (a majority of scans show 100% even when accuracy is
  reduced), so fragment-ion scores would be the correct choice, but the reverse (using
  ms1PrecSignal alone) is the mistake.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java standalone tool that computes the fragment ion intensity QC scores, quantifies
    reporter ions, and writes the isofrag/isolab/isopep/isopep_extra CSV outputs
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: Converts instrument raw files to the mzML input format required by IsobaricQuant
- name: Comet
  role: Search engine whose output is converted into the PSM hits file consumed for
    fragment ion matching
- name: Java 17
  role: Runtime dependency required to execute the IsobaricQuant jar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:69fd56b0fd01edabb4ea1cbcdcd624ddf1ffd8c10591719435444b379a36e5b1
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
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fragment-ion-intensity-scoring

## Summary

Compute IsobaricQuant fragment-ion-based QC scores (peptideIntensityScore and peptideTopXIntensityFromTotalScore) by matching b/y fragment ions, including neutral losses 63.98, 18.01 and 17.03, against MS2/MS3 scan peaks to quantify the fraction of scan intensity explained by the target peptide, enabling filtering of scans with co-isolation (ratio compression).

## When to use

Use when iTRAQ/TMT/TMTpro-labeled PSMs need interference QC scores computed per scan from b/y fragment ion intensity fractions in mzML data, to filter compressed reporter ion quantifications.

## When NOT to use

- Input is a protein-level feature table or already-filtered quantification matrix — the scores must be computed per scan prior to protein-level quantification.
- Data are label-free or use non-isobaric labeling (the score calculations revolve around iTRAQ/TMT/TMTpro reporter and complementary ion series).
- Only MS1 precursor purity assessment is needed — ms1PrecSignal is a limited predictor of reporter ion interference (a majority of scans show 100% even when accuracy is reduced), so fragment-ion scores would be the correct choice, but the reverse (using ms1PrecSignal alone) is the mistake.

## Inputs

- mzML file of the MS run (converted from raw via msconvert, e.g. with peakPicking and zeroSamples removeExtra filters)
- CSV file of peptide spectral matches (PSM) from a search engine such as Comet, with columns Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z
- User-supplied text configuration file (scanLevel, PPMTolerance 20, topXNum 10, label m/z definitions)
- Optional reporter ion isotopic impurity correction CSV

## Outputs

- isofrag.csv (fragment-level annotations)
- isolab.csv (isobaric label intensities per scan, non-negative, matched within ppm tolerance)
- isopep.csv (per-peptide-scan quantification)
- isopep_extra.csv (per-scan QC scores including peptideIntensityScore and peptideTopXIntensityFromTotalScore in [0,1])

## How to apply

Parse the mzML scan file and the comma-separated peptide hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) together with a user-supplied config (scanLevel, PPMTolerance 20, MS2PPMTolerance 20, topXNum 10). For each scan, match theoretical b/y fragment ions of the assigned peptide — including neutral losses of 63.98, 18.01 and 17.03 — to observed peaks within the ppm tolerance, and exclude MS1 non-fragmented precursor M+H peaks, TMT reporter ions, and TMTpro complementary ion cluster series from MS2 score calculations. Compute peptideIntensityScore = totalFragmIonsIntensity/totalIntensity and peptideTopXIntensityFromTotalScore = topXFragmIonsIntensity/totalIntensity with topX = 10, chosen to equal the number of SPS-MS3 selection notches. For MS3-level data, recommended filtering cutoffs are 0.25 for peptideIntensityScore and 0.15 for peptideTopXIntensityFromTotalScore; the 25th percentile of the score distribution is a good starting point for dataset-specific cutoff optimization. Write results to the isopep.csv/isopep_extra.csv outputs and validate that score values fall in [0,1].

## Related tools

- **IsobaricQuant** (Java standalone tool that computes the fragment ion intensity QC scores, quantifies reporter ions, and writes the isofrag/isolab/isopep/isopep_extra CSV outputs) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (Converts instrument raw files to the mzML input format required by IsobaricQuant)
- **Comet** (Search engine whose output is converted into the PSM hits file consumed for fragment ion matching)
- **Java 17** (Runtime dependency required to execute the IsobaricQuant jar)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf HYpro16.mzML -h HYpro16_psm.csv -o out/
```

## Evaluation signals

- peptideIntensityScore and peptideTopXIntensityFromTotalScore columns are present in isopep_extra.csv and fall within the valid range [0,1].
- Label intensities in isolab.csv are non-negative and each matched peak was found within +/- PPMTolerance (20 ppm) of the label m/z.
- topX = 10 fragment ions are used for peptideTopXIntensityFromTotalScore, matching the number of SPS-MS3 selection notches.
- MS3-level filtering at the recommended cutoffs (0.25 peptideIntensityScore, 0.15 peptideTopXIntensityFromTotalScore) decreases protein CVs without substantially reducing the number of observed proteins.
- Excluded-peak logic is confirmed: MS1 non-fragmented precursor M+H, TMT reporter ions and TMTpro complementary ion clusters contribute zero to MS2 score calculations.

## Limitations

- Cutoff values are dataset-dependent; the optimal score and cutoff must be re-optimized per dataset (the 25th percentile is a good starting value).
- Filtering out interfered quantifications improves accuracy and precision but significantly decreases peptide observations, potentially exacerbating missing values across multiple labeled sample sets.
- The identity of co-isolated peptides and their reporter ion release efficiency is unknown, so ratio decompression is not possible with these scores.
- IsobaricQuant supports iTRAQ up to 8-plex, TMT up to 11-plex and TMTpro up to 16-plex only.

## Evidence

- [discussion] For peptideTopXIntensityFromTotalScore, we set topX =10 to equal the number of SPS-MS3 selection notches: "For peptideTopXIntensityFromTotalScore, we set topX =10 to equal the number of SPS-MS3 selection notches"
- [methods] For all score calculations in MS2, the following peaks are excluded: MS1 non-fragmented precursor M+H, TMT reporter ions, TMTpro complementary ion clusters: [evidence span withheld — anchor status: not_found]
- [abstract] its QC scores can successfully filter out scans with reduced quantitative accuracy at MS2 and MS3 levels, removing inaccurate peptide quantifications and decreasing protein CVs: "its QC scores can successfully filter out scans with reduced quantitative accuracy at MS2 and MS3 levels, removing inaccurate peptide quantifications and decreasing protein CVs"
- [readme] java -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>: "Or it can be used via command line: java -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>"
- [discussion] we expect that both optimal score and cutoff will be dataset-dependent: "In the three datasets analyzed in this manuscript we observed slightly different score distributions. Thus, we expect that both optimal score and cutoff will be dataset-dependent."
