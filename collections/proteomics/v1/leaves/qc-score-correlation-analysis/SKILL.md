---
name: qc-score-correlation-analysis
description: Use when you have isobarically-labeled (e.g. TMTpro 16-plex) PSMs with
  known mixing ratios (ground truth) and per-scan IsobaricQuant QC scores, and you
  need to determine which score predicts compression or reduced reporter-ion ratio
  accuracy.
when_to_use_negative:
- The dataset has no ground-truth mixing ratio (e.g. a real biological PISA dataset),
  since ratio accuracy cannot be computed without expected ratios
- Input PSMs have not been quantified by IsobaricQuant yet — there are no per-PSN
  QC scores to correlate until the quantification step is run
- You only need a filtering step with a known-good score for the data type, rather
  than determining which score/cutoff is optimal — the paper notes optimal score and
  cutoff are dataset-dependent
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Generates per-PSN QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore,
    peptideTopXIntensityFromTotalScore) via CLI quantification of TMTpro-labeled MS2/MS3
    data
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar in CLI mode
- name: Comet
  role: Search engine whose PSM output is converted into the hits (CSV) input required
    by IsobaricQuant
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:512fad4bc5fba10b251229b3182938141967c09fecae5b1cf8edcd9607e41534
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Java 17
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# qc-score-correlation-analysis

## Summary

A validation analysis that correlates IsobaricQuant QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) with quantitative ratio accuracy in isobaric mass tag data, to identify which score best predicts reduced accuracy at MS2 vs MS3 levels.

## When to use

Use when you have isobarically-labeled (e.g. TMTpro 16-plex) PSMs with known mixing ratios (ground truth) and per-scan IsobaricQuant QC scores, and you need to determine which score best predicts compression or reduced reporter-ion ratio accuracy.

## When NOT to use

- The dataset has no ground-truth mixing ratio (e.g. a real biological PISA dataset), since ratio accuracy cannot be computed without expected ratios
- Input PSMs have not been quantified by IsobaricQuant yet — there are no per-PSN QC scores to correlate until the quantification step is run
- You only need a filtering step with a known-good score for the data type, rather than determining which score/cutoff is optimal — the paper notes optimal score and cutoff are dataset-dependent

## Inputs

- mzML file of the MS run (converted from raw via msconvert, e.g. with peakPicking true 1-, --32, --zlib filters)
- CSV file of peptide spectral matches (PSM) from a search engine such as Comet
- IsobaricQuant text configuration file specifying TMTpro labels, PPMTolerance 20, topXNum 10, and MS2/MS3 scan levels
- Dataset with known mixing ratios (ground truth, e.g. yeast 20:1) such as the HYpro16 dataset

## Outputs

- isopep_extra.csv with per-peptide QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore)
- Per-PSM ratio-accuracy values (deviation of observed reporter-ion ratios from expected 20:1 ground truth)
- score_accuracy_summary.csv with one row per QC score per scan level (MS2 and MS3), including slopes and correlation coefficients

## How to apply

Run IsobaricQuant (Java 17, CLI mode: java -jar IsobaricQuant.jar -c config -mzf mzfile -h hits -o outfolder) on a dataset with known mixing ratios, such as a yeast 20:1 ratio experiment, at both MS2 and MS3 scan levels with TMTpro labels, PPMTolerance 20 and topXNum 10. Parse the isopep_extra.csv output to extract per-peptide ms1PrecSignal (precursor signal), msnTotalSignal, peptideIntensityScore, and peptideTopXIntensityFromTotalScore. Compute observed reporter-ion ratios per PSM and compare them to the expected 20:1 ratio to quantify per-PSM ratio accuracy (deviation from ground truth). Bin precursors by each QC score and correlate score bins with ratio accuracy, computing slopes and correlation coefficients for MS2 and MS3 separately, and write the summary to score_accuracy_summary.csv with one row per score per scan level. Verify the characteristic patterns: ms1PrecSignal saturation (majority of scans at 100% despite reduced accuracy), msnTotalSignal as the most extreme-slope/highest-correlation score at MS2, and peptideIntensityScore (cutoff 0.25) / peptideTopXIntensityFromTotalScore (cutoff 0.15, topX = 10) as the best MS3 predictors.

## Related tools

- **IsobaricQuant** (Generates per-PSN QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) via CLI quantification of TMTpro-labeled MS2/MS3 data) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Runtime required to execute the IsobaricQuant jar in CLI mode)
- **Comet** (Search engine whose PSM output is converted into the hits (CSV) input required by IsobaricQuant)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf HYpro16.mzML -h comet_hits.csv -o outfolder
```

## Evaluation signals

- ms1PrecSignal shows saturation: a majority of scans report 100% even when quantitative accuracy is reduced — this pattern must reproduce
- msnTotalSignal exhibits the most extreme slope and highest correlation with ratio accuracy at the MS2 level
- peptideIntensityScore (cutoff 0.25) and peptideTopXIntensityFromTotalScore (cutoff 0.15, topX = 10) emerge as the best predictors at the MS3 level
- score_accuracy_summary.csv contains one row per QC score per scan level (MS2 and MS3) with slopes and correlation coefficients populated
- Observed reporter-ion ratios deviate measurably from the expected 20:1 ratio for precursors flagged as interfered

## Limitations

- ms1PrecSignal has the lowest predictive value of reduced quantitative accuracy among IsobaricQuant QC scores, explained by up to 60% of precursors showing 100% despite reduced accuracy
- Optimal QC score and cutoff are dataset-dependent — score distributions differed across the three datasets analyzed in the manuscript
- The identity of co-isolated peptides and their reporter ion release efficiency remain unknown, preventing ratio decompression
- Filtering interfered quantifications improves accuracy and precision but significantly decreases peptide observations (mitigable via fractionation)

## Evidence
