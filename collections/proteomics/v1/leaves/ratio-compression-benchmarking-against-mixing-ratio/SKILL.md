---
name: ratio-compression-benchmarking-against-mixing-ratio
description: Use when isobarically-labeled (TMT/TMTpro/iTRAQ) data contain samples
  mixed at a known ratio and you need to determine which QC score predicts reduced
  quantitative accuracy (ratio compression) at MS2 vs MS3.
when_to_use_negative:
- The dataset has no known mixing ratio or spike-in ground truth — deviation from
  expected ratios cannot be computed
- Data are not isobarically-labeled (e.g. label-free or SILAC quantification) — IsobaricQuant
  QC scores and reporter-ion ratios do not apply
- The goal is protein-level differential analysis rather than auditing which precursor-level
  QC score predicts quantification inaccuracy
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IsobaricQuant
  role: Computes per-scan QC scores and reporter-ion quantification from mzML + PSM
    hits; run in CLI mode at MS2 and MS3 levels to obtain isopep_extra.csv
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search engine whose PSM output is converted into the hits file consumed by
    IsobaricQuant
- name: msconvert
  role: Converts instrument raw files to mzML with recommended filters (peakPicking
    true 1-, zeroSamples removeExtra, --32, --zlib) before IsobaricQuant
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:74165a27d0202f27a2f7a9241ab10ce99d7b20ff390355ab71162982761bbf68
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
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ratio-compression-benchmarking-against-mixing-ratio

## Summary

Benchmarks IsobaricQuant QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) by correlating score bins with per-PSM deviation of observed reporter-ion ratios from a known mixing ratio (e.g. yeast 20:1), separately at MS2 and MS3 levels.

## When to use

Use when isobarically-labeled (TMT/TMTpro/iTRAQ) data contain samples mixed at a known ratio and you need to determine which QC score best predicts reduced quantitative accuracy (ratio compression) at MS2 vs MS3.

## When NOT to use

- The dataset has no known mixing ratio or spike-in ground truth — deviation from expected ratios cannot be computed
- Data are not isobarically-labeled (e.g. label-free or SILAC quantification) — IsobaricQuant QC scores and reporter-ion ratios do not apply
- The goal is protein-level differential analysis rather than auditing which precursor-level QC score predicts quantification inaccuracy

## Inputs

- mzML file of the isobarically-labeled MS run (converted via msconvert) with known sample mixing ratio (e.g. yeast 20:1 TMTpro)
- CSV of PSM hits from a search engine such as Comet
- IsobaricQuant text configuration file (TMTpro labels, PPMTolerance 20, topXNum 10, MS2 and MS3 scan levels)
- Optional reporter ion isotopic distribution CSV for impurity correction
- Ground-truth expected mixing ratio (e.g. 20:1) for ratio-accuracy calculation

## Outputs

- isopep_extra.csv with per-PSN/per-peptide QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) and observed reporter-ion intensities
- Per-PSM ratio-accuracy values (deviation of observed reporter-ion ratios from the expected 20:1 ratio)
- Correlation statistics (slopes and correlation coefficients) per QC score per scan level, e.g. score_accuracy_summary.csv with one row per score per scan level

## How to apply

Run IsobaricQuant (Java 17, CLI: java -jar IsobaricQuant.jar -c config -mzf mzfile -h hits -o outfolder) on the ground-truth mixed dataset, e.g. the HYpro16 yeast 20:1 dataset with TMTpro labels, PPMTolerance 20, and topXNum 10, at both MS2 and MS3 scan levels. Parse isopep_extra.csv for ms1PrecSignal, msnTotalSignal, peptideIntensityScore, and peptideTopXIntensityFromTotalScore per peptide, compute observed reporter-ion ratios, and quantify per-PSM ratio accuracy as deviation from the expected 20:1 ratio. Bin precursors by each QC score and correlate bins with ratio accuracy (slopes and correlation coefficients) for MS2 and MS3 separately. Validate the characteristic patterns: ms1PrecSignal saturates at 100% for most scans despite reduced accuracy (lowest predictive value, explaining up to 60% of precursors), msnTotalSignal shows the most extreme slope and highest correlation at MS2 (recommended cutoff 1e6), while peptideIntensityScore (cutoff 0.25) and peptideTopXIntensityFromTotalScore (cutoff 0.15, topX=10 to match SPS-MS3 notches) are the best MS3 predictors. Write one summary row per score per scan level (e.g. score_accuracy_summary.csv) and remember that the 25th percentile is a good starting point for dataset-specific cutoff optimization.

## Related tools

- **IsobaricQuant** (Computes per-scan QC scores and reporter-ion quantification from mzML + PSM hits; run in CLI mode at MS2 and MS3 levels to obtain isopep_extra.csv) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Runtime required to execute the IsobaricQuant jar) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search engine whose PSM output is converted into the hits file consumed by IsobaricQuant)
- **msconvert** (Converts instrument raw files to mzML with recommended filters (peakPicking true 1-, zeroSamples removeExtra, --32, --zlib) before IsobaricQuant)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf HYpro16.mzML -h comet_hits.csv -o outfolder
```

## Evaluation signals

- ms1PrecSignal shows saturation: a majority of scans display 100% even where quantitative accuracy is reduced, and it has the lowest predictive value of reduced accuracy (explaining up to 60% of precursors)
- msnTotalSignal yields the most extreme slope and highest correlation with ratio accuracy among MS2-level scores; recommended MS2 cutoff of 1e6 gives good filtering performance
- At MS3, peptideIntensityScore (cutoff 0.25) and peptideTopXIntensityFromTotalScore (cutoff 0.15, topX=10 equal to SPS-MS3 notches) are the best predictors of ratio accuracy
- Correlation coefficients and slopes are computed separately for MS2 and MS3 and the summary contains one row per score per scan level
- Filtering by the best score reduces reporter-ion ratio deviation and decreases protein CVs without greatly reducing the number of observed proteins

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- Filtering improves accuracy and precision but significantly decreases peptide observations, exacerbating missing values across multiple labeled sample sets
- The identity of co-isolated peptides and their reporter ion release efficiency are unknown, so ratio decompression cannot be directly corrected
- Results depend on the ground-truth mixing design (e.g. 20:1 yeast); findings on one dataset may not transfer without re-benchmarking

## Evidence
