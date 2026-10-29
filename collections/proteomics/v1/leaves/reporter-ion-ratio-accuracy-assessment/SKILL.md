---
name: reporter-ion-ratio-accuracy-assessment
description: Use when quantifying isobarically-labeled peptides with a known mixing
  ratio (e.g., yeast 20:1 ratio, TMTpro 16-plex) and you need to quantify ratio compression
  per PSM/PSN and identify which IsobaricQuant QC score predicts reduced quantitative
  accuracy.
when_to_use_negative:
- The dataset lacks a known ground-truth mixing ratio (e.g., a discovery dataset like
  PISA staurosporine), so no per-PSM ratio deviation from expected values can be computed
- The data are not isobarically labeled (iTRAQ/TMT/TMTpro) or are label-free, so IsobaricQuant
  reporter-ion scores and ratio compression are not applicable
- The goal is protein-level differential analysis rather than auditing scan-level
  quantitative accuracy of reporter ions
edam_operation: http://edamontology.org/operation_3635
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: IsobaricQuant
  role: Computes per-PSN QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore,
    peptideTopXIntensityFromTotalScore) and reporter-ion intensities at MS2/MS3 levels
    used for ratio-accuracy correlation
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search engine whose PSM output file is converted into the hits input required
    by IsobaricQuant
- name: Java 17
  role: Required runtime environment to execute the IsobaricQuant jar in CLI mode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:6d2ba6556a351c65bba393147e11df943c49e6177cd38aa2f1dfd2b225f9e78f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Java 17
  edam_operation: http://edamontology.org/operation_3635
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# reporter-ion-ratio-accuracy-assessment

## Summary

Assesses quantitative ratio accuracy of isobaric mass tag (TMT/TMTpro/iTRAQ) reporter-ion measurements by comparing observed reporter-ion ratios to a known ground-truth mixing ratio (e.g., 20:1), and correlates these deviations with IsobaricQuant QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) at MS2 and MS3 scan levels.

## When to use

Use when quantifying isobarically-labeled peptides with a known mixing ratio (e.g., yeast 20:1 ratio, TMTpro 16-plex) and you need to quantify ratio compression per PSM/PSN and identify which IsobaricQuant QC score best predicts reduced quantitative accuracy.

## When NOT to use

- The dataset lacks a known ground-truth mixing ratio (e.g., a discovery dataset like PISA staurosporine), so no per-PSM ratio deviation from expected values can be computed
- The data are not isobarically labeled (iTRAQ/TMT/TMTpro) or are label-free, so IsobaricQuant reporter-ion scores and ratio compression are not applicable
- The goal is protein-level differential analysis rather than auditing scan-level quantitative accuracy of reporter ions

## Inputs

- mzML file of the isobarically-labeled MS run (converted from raw via msconvert)
- CSV file of peptide spectral matches (PSMs) from a search engine such as Comet
- IsobaricQuant text configuration file (e.g., TMTpro 16-plex, PPMTolerance 20, topXNum 10)
- Ground-truth expected mixing ratio (e.g., yeast 20:1) for ratio-accuracy calculation
- isopep_extra.csv IsobaricQuant output containing per-peptide QC scores

## Outputs

- Per-PSM observed vs. expected reporter-ion ratio deviations (ratio accuracy)
- Score-accuracy correlation summary (slopes and correlation coefficients per QC score per scan level, e.g., score_accuracy_summary.csv)
- Ranked assessment of QC scores as predictors of reduced quantitative accuracy at MS2 vs. MS3 levels

## How to apply

Run IsobaricQuant (Java 17) in CLI mode on the mzML file and search-engine hits file to obtain per-PSN QC scores, writing per-peptide scores into the isopep_extra.csv output. Parse ms1PrecSignal, msnTotalSignal, peptideIntensityScore, and peptideTopXIntensityFromTotalScore for each peptide, then compute observed reporter-ion ratios and compare them to the expected ground-truth ratio (e.g., 20:1) to quantify per-PSM deviation from the expected ratio. Bin precursors by each QC score and correlate score bins with ratio accuracy, computing slopes and correlation coefficients separately for MS2 and MS3 scans. Verify characteristic patterns: ms1PrecSignal saturation (majority of scans at 100% despite reduced accuracy), msnTotalSignal as the most extreme-slope/highest-correlation score at MS2, and peptideIntensityScore (cutoff 0.25) and peptideTopXIntensityFromTotalScore (cutoff 0.15, topX = 10) as the best MS3 predictors. Note that the majority of scans show an ms1PrecSignal of 100% even though they are still heavily impacted by reduced quantitative accuracy, making ms1PrecSignal the lowest-value predictor.

## Related tools

- **IsobaricQuant** (Computes per-PSN QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) and reporter-ion intensities at MS2/MS3 levels used for ratio-accuracy correlation) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search engine whose PSM output file is converted into the hits input required by IsobaricQuant)
- **Java 17** (Required runtime environment to execute the IsobaricQuant jar in CLI mode)

## Examples

```
java -jar IsobaricQuant.jar -c config -mzf mzfile -h hits -o outfolder
```

## Evaluation signals

- isopep_extra.csv contains ms1PrecSignal, msnTotalSignal, peptideIntensityScore, and peptideTopXIntensityFromTotalScore values for every quantified PSM
- ms1PrecSignal shows saturation: a majority of scans report 100% even though quantitative accuracy is reduced
- At MS2 level, msnTotalSignal exhibits the most extreme slope and highest correlation with ratio accuracy
- At MS3 level, peptideIntensityScore (cutoff 0.25) and peptideTopXIntensityFromTotalScore (cutoff 0.15, topX = 10) are the best-performing predictors
- Observed reporter-ion ratios deviate from the expected 20:1 ground truth by a quantifiable per-PSM amount, summarized in one row per score per scan level (e.g., score_accuracy_summary.csv)

## Limitations

- ms1PrecSignal has the lowest predictive value of reduced quantitative accuracy because up to 60% of precursors show saturation at 100% despite interference
- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a good starting value for cutoff optimization, and cutoffs must be adjusted per dataset
- Filtering out interfered quantifications improves accuracy and precision but significantly decreases peptide observations, potentially exacerbating missing values across multiple labeled sample sets; the identity of co-isolated peptides and their reporter ion release efficiency remain unknown, preventing ratio decompression

## Evidence
