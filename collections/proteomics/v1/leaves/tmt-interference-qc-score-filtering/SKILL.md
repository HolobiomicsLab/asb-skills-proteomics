---
name: tmt-interference-qc-score-filtering
description: Use when isobarically-labeled (TMT/TMTpro/iTRAQ) MS2 or RTS-MS3 reporter-ion
  quantifications suffer precursor interference (ratio compression) and you want to
  filter inaccurate scans with QC score cutoffs before protein-level quantification.
when_to_use_negative:
- Data are label-free or otherwise not isobarically tagged (iTRAQ/TMT/TMTpro) — IsobaricQuant
  scores address reporter-ion co-isolation interference
- Quantification is already at RTS-MS3 acquisition with only ~2 PSMs per protein,
  where filtering possibilities are severely limited
- You need every peptide observation and cannot tolerate increased missing values
  — stringent filtering significantly decreases peptide observations
edam_operation: http://edamontology.org/operation_3705
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IsobaricQuant
  role: Computes reporter ion intensities and QC scores for TMT/iTRAQ peptides and
    performs the scan-level filtering
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: Converts vendor raw files to mzML with peak-picking filters as IsobaricQuant
    input
- name: Comet
  role: Search engine whose PSM output is converted to the IsobaricQuant hits CSV
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:b6786bd1f7ed998854c1a7972c66097b8bb61c90aaa0606d3635ccf6d65298be
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
  edam_operation: http://edamontology.org/operation_3705
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# tmt-interference-qc-score-filtering

## Summary

Filter co-isolation-interfered TMT/TMTpro/iTRAQ peptide quantifications using IsobaricQuant QC scores (e.g. peptideTopXIntensityFromTotalScore, peptideIntensityScore, msnTotalSignal) applied prior to protein-level aggregation, improving quantitative accuracy and target sensitivity while preserving protein depth.

## When to use

Use when isobarically-labeled (TMT/TMTpro/iTRAQ) MS2 or RTS-MS3 reporter-ion quantifications suffer precursor interference (ratio compression) and you want to filter inaccurate scans with QC score cutoffs before protein-level quantification.

## When NOT to use

- Data are label-free or otherwise not isobarically tagged (iTRAQ/TMT/TMTpro) — IsobaricQuant scores address reporter-ion co-isolation interference
- Quantification is already at RTS-MS3 acquisition with only ~2 PSMs per protein, where filtering possibilities are severely limited
- You need every peptide observation and cannot tolerate increased missing values — stringent filtering significantly decreases peptide observations

## Inputs

- mzML file of the MS run (converted from raw with msconvert, e.g. peakPicking true 1-, zeroSamples removeExtra)
- CSV hits file of PSMs from a search engine such as Comet/Percolator (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- IsobaricQuant text config file (e.g. TMTpro isoMethod, 20 ppm PPMTolerance/MS1PPMTolerance/MS2PPMTolerance, scanLevel MS2 or RTS-MS3, topXNum = 10)
- Optional reporter ion isotopic impurity correction CSV

## Outputs

- isofrag.csv, isolab.csv, isopep.csv and isopep_extra.csv with per-scan/per-peptide reporter intensities and QC scores
- Filtered peptide quantification table prior to protein-level aggregation (e.g. 4,713 filtered proteins vs 5,380 unfiltered MS2-level)
- Improved downstream target lists (e.g. PISA significant staurosporine targets: 4→7 at 1 uM, 17→22 at 20 uM)

## How to apply

Run IsobaricQuant (Java 17, CLI mode) on mzML files with a search-engine hits file to obtain per-scan QC scores and reporter intensities (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv). Choose the QC score appropriate to the acquisition level: for MS2 data use msnTotalSignal (recommended cutoff 1e6); for MS3/SPS data use peptideIntensityScore (cutoff 0.25) or peptideTopXIntensityFromTotalScore with topX = 10 to match the SPS-MS3 selection notches (cutoff 0.15). Derive the cutoff from the score distribution's 25th percentile as a starting value, since optimal score and cutoff are dataset-dependent. Apply filtering directly prior to protein-level quantification so protein inference is not altered, then re-run the downstream analysis (e.g. PISA thermal-shift significance) to confirm improved sensitivity without large protein-depth loss.

## Related tools

- **IsobaricQuant** (Computes reporter ion intensities and QC scores for TMT/iTRAQ peptides and performs the scan-level filtering) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (Converts vendor raw files to mzML with peak-picking filters as IsobaricQuant input)
- **Comet** (Search engine whose PSM output is converted to the IsobaricQuant hits CSV)
- **Java 17** (Runtime required to execute the IsobaricQuant jar)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o output/
```

## Evaluation signals

- The four IsobaricQuant CSV outputs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) exist and parse correctly
- Protein quantification counts match reported targets: 5,380 unfiltered MS2-level, 4,033 RTS-MS3-level, 4,713 after MS2 filtering
- Downstream significant-target counts improve after filtering: PISA staurosporine 4→7 at 1 uM and 17→22 at 20 uM
- Protein CVs decrease after removing interfered scans while the number of observed proteins is largely preserved
- Cutoff derivation is documented from the score distribution (25th percentile starting value; e.g. peptideTopXIntensityFromTotalScore = 0.15 with topX = 10)

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- Filtering improves accuracy and precision at the cost of significantly decreased peptide observations, potentially exacerbating missing values across multiple labeled sets
- ms1PrecSignal is a poor predictor of interference since a majority of scans show 100% even when accuracy is reduced
- Identity of co-isolated peptides and their reporter ion release efficiency remain unknown, preventing ratio decompression

## Evidence
