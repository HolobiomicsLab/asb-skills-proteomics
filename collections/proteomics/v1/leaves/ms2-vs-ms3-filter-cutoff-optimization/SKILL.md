---
name: ms2-vs-ms3-filter-cutoff-optimization
description: Use when filtering isobarically-labeled (iTRAQ/TMT/TMTpro) peptide quantifications
  for co-isolation interference and you must choose which IsobaricQuant QC score and
  threshold to apply, differently for MS2- and MS3-level data.
when_to_use_negative:
- The dataset's score distributions have not been examined — cutoffs like 1e6 (msnTotalSignal,
  MS2) or 0.25/0.15 (MS3) are not universally applicable and must be adjusted per
  dataset
- The acquisition method limits PSMs per protein (e.g., the PISA RTS-MS3 method acquired
  a maximum of 2 PSMs per protein), leaving too few peptides for filtering to help
- Data are not isobarically-labeled (iTRAQ/TMT/TMTpro) or not quantified with IsobaricQuant
  QC scores — the score semantics and cutoffs do not transfer
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Quantifies reporter ions and computes QC scores (ms1PrecSignal, msnTotalSignal,
    peptideIntensityScore, peptideTopXIntensityFromTotalScore) used to filter interfered
    scans at MS2 and MS3 levels
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar via GUI or command line
- name: Comet
  role: Search engine whose PSM output file is converted to the hits CSV input required
    by IsobaricQuant
  repo: https://uwpr.github.io/Comet/
- name: msconvert
  role: Converts instrument raw files to mzML with recommended filters before IsobaricQuant
    quantification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:f69c7e929e8d6a4d990bc2c42b761f37f2851a6f2350eb5c0ba3fad2fd7dae0f
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
yes_rate: 1.0
n_claims: 15
---

# ms2-vs-ms3-filter-cutoff-optimization

## Summary

A method for selecting IsobaricQuant QC scores and filtering cutoffs that separately optimize removal of interfered, quantitatively inaccurate scans at MS2 versus MS3 levels, using score-accuracy relationships from ratio-compressed standards and dataset-specific percentile-based cutoff tuning.

## When to use

Use when filtering isobarically-labeled (iTRAQ/TMT/TMTpro) peptide quantifications for co-isolation interference and you must choose which IsobaricQuant QC score and threshold to apply, differently for MS2- and MS3-level data.

## When NOT to use

- The dataset's score distributions have not been examined — cutoffs like 1e6 (msnTotalSignal, MS2) or 0.25/0.15 (MS3) are not universally applicable and must be adjusted per dataset
- The acquisition method limits PSMs per protein (e.g., the PISA RTS-MS3 method acquired a maximum of 2 PSMs per protein), leaving too few peptides for filtering to help
- Data are not isobarically-labeled (iTRAQ/TMT/TMTpro) or not quantified with IsobaricQuant QC scores — the score semantics and cutoffs do not transfer

## Inputs

- mzML file converted from instrument raw data (e.g., via msconvert with peakPicking true 1- and zeroSamples removeExtra filters)
- CSV file of peptide spectral matches (PSMs) from a search engine such as Comet
- IsobaricQuant text configuration file (labels, PPMTolerance, scan level MS2/MS3, topXNum)
- Optional reporter ion isotopic impurity correction CSV
- Ground-truth mixing ratio (e.g., yeast 20:1 TMTpro standard) for validating score-accuracy relationships

## Outputs

- Per-PSN QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) in isopep_extra.csv
- Filtered PSM/peptide quantification tables at MS2 and MS3 levels after applying level-specific score cutoffs
- Score-accuracy relationship summary (slopes and correlation coefficients per score per scan level, e.g., score_accuracy_summary.csv)
- Protein-level quantifications with reduced CVs after filtering

## How to apply

Run IsobaricQuant (Java 17, CLI: java -jar IsobaricQuant.jar -c config -mzf mzfile -h hits -o outfolder) on the mzML and PSM hit file to obtain per-PSN QC scores from isopep_extra.csv. Bin precursors by each QC score and correlate score bins with ratio accuracy against a ground-truth mixing ratio (e.g., a 20:1 yeast ratio standard) to identify which scores actually predict reduced quantitative accuracy at each scan level. At MS2, msnTotalSignal is the strongest predictor, with a recommended cutoff of 1e6; at MS3, peptideIntensityScore (cutoff 0.25) and peptideTopXIntensityFromTotalScore (cutoff 0.15, with topX = 10 to equal the number of SPS-MS3 selection notches) give the best filtering. Start cutoff optimization from the 25th percentile of each score's distribution and tune per dataset, since score distributions and optimal cutoffs are dataset-dependent. Apply filtering directly prior to protein-level quantification so protein inference is not altered, and inspect the resulting trade-off between improved accuracy/precision (decreased protein CVs) and loss of peptide observations.

## Related tools

- **IsobaricQuant** (Quantifies reporter ions and computes QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) used to filter interfered scans at MS2 and MS3 levels) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Runtime required to execute the IsobaricQuant jar via GUI or command line)
- **Comet** (Search engine whose PSM output file is converted to the hits CSV input required by IsobaricQuant) — https://uwpr.github.io/Comet/
- **msconvert** (Converts instrument raw files to mzML with recommended filters before IsobaricQuant quantification)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf HYpro16.mzML -h comet_hits.csv -o outfolder
```

## Evaluation signals

- ms1PrecSignal should show saturation behavior: a majority of scans at 100% even where quantitative accuracy is reduced — it is the weakest predictor and should not be the primary filter
- At MS2, msnTotalSignal should show the most extreme slope and highest correlation with ratio accuracy; the 1e6 cutoff should remove inaccurate quantifications and decrease protein CVs
- At MS3, filtering with peptideIntensityScore (0.25) or peptideTopXIntensityFromTotalScore (0.15, topX=10) should improve ratio accuracy while preserving the number of observed proteins at the protein level
- Filtering should visibly decrease peptide observations (missing values increase) — absence of any peptide loss suggests the cutoff is too lenient
- Slopes/correlations should be recomputed separately for MS2 and MS3; pooled analysis across scan levels is incorrect

## Limitations

- Optimal QC score and cutoff are dataset-dependent; score distributions differed across the three datasets analyzed and the 25th percentile is only a starting point for optimization
- Filtering improves accuracy and precision at the cost of significantly decreased peptide observations, exacerbating missing values when multiple labeled sample sets are measured
- ms1PrecSignal has the lowest predictive value of reduced quantitative accuracy because up to 60% of precursors show 100% purity despite reduced accuracy
- The identity of co-isolated peptides and their reporter ion release efficiency are unknown, preventing ratio decompression
- RTS-MS3 methods with a maximum of 2 PSMs per protein significantly limit filtering possibilities

## Evidence
