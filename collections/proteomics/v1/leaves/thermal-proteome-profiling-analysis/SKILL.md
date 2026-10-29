---
name: thermal-proteome-profiling-analysis
description: Use when reprocessing TMTpro isobaric-labeled PISA/thermal-proteome raw
  data (e.g., PRIDE PXD014369) to compare unfiltered vs. QC-filtered protein quantification
  and count significant thermal-shift drug targets.
when_to_use_negative:
- Input data were acquired with RTS-MS3 limited to a maximum of 2 PSMs per protein,
  which significantly limits the possibility of filtering peptides
- The goal is cross-run normalization or downstream statistical modeling of an existing
  protein-level table rather than scan-level isobaric quantification and QC filtering
- The labeling scheme exceeds IsobaricQuant support (iTRAQ up to 8-plex, TMT up to
  11-plex, TMTpro up to 16-plex) or the data are not isobarically labeled
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java-based quantification, visualization, and QC-score filtering of TMTpro-labeled
    PISA peptides from mzML plus hits-file input; run via CLI to generate isofrag/isolab/isopep
    CSVs
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: Converts PRIDE PXD014369 raw instrument files to mzML for IsobaricQuant input
  repo: https://isobaricquant.readthedocs.io/en/latest/
- name: Comet
  role: Search engine whose output is converted into the IsobaricQuant hits file format
- name: Percolator
  role: Validates/weights PSMs before conversion to the IsobaricQuant hits CSV
- name: Java 17
  role: Required runtime to execute the IsobaricQuant jar
  repo: https://github.com/Villen-Lab/isobaricquant
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:6e8e8a78d72ed73f94044b5e3a6d1cc3f178e3901f4d2f6be5c523d9aeb64bba
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# thermal-proteome-profiling-analysis

## Summary

Reproduces the IsobaricQuant-based PISA thermal-shift workflow (PXD014369, staurosporine), in which TMTpro-labeled MS2- and RTS-MS3-acquired PISA data are quantified, QC-score-filtered (PeptideTopXIntensityFromTotalScore, cutoff 0.15, topX=10), and aggregated to protein level to improve sensitivity of kinase-inhibitor target identification.

## When to use

Use when reprocessing TMTpro isobaric-labeled PISA/thermal-proteome raw data (e.g., PRIDE PXD014369) to compare unfiltered vs. QC-filtered protein quantification and count significant thermal-shift drug targets.

## When NOT to use

- Input data were acquired with RTS-MS3 limited to a maximum of 2 PSMs per protein, which significantly limits the possibility of filtering peptides
- The goal is cross-run normalization or downstream statistical modeling of an existing protein-level table rather than scan-level isobaric quantification and QC filtering
- The labeling scheme exceeds IsobaricQuant support (iTRAQ up to 8-plex, TMT up to 11-plex, TMTpro up to 16-plex) or the data are not isobarically labeled

## Inputs

- Raw instrument files for the PISA staurosporine experiment (1 and 20 uM) from PRIDE PXD014369
- mzML files converted from raw via msconvert
- Comet/Percolator search output converted to the IsobaricQuant hits CSV (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- IsobaricQuant text configuration file (isoMethod TMTpro, 20 ppm tolerances, scanLevel MS2 or RTS-MS3, topXNum=10)

## Outputs

- IsobaricQuant output CSVs: isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv
- Protein-level reporter intensity quantification (MS2 target depth 5,380 proteins; RTS-MS3 target 4,033 proteins; filtered target 4,713 proteins)
- PISA thermal-shift significance results: significant staurosporine targets (1 uM: 4 unfiltered to 7 filtered; 20 uM: 17 unfiltered to 22 filtered)

## How to apply

Download the PISA staurosporine raw files (1 and 20 uM) from PRIDE PXD014369 and convert to mzML with msconvert (peakPicking true 1-, zeroSamples removeExtra, 32-bit, zlib). Search spectra with Comet, validate PSMs with Percolator, and convert to the IsobaricQuant hits CSV (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z). Run IsobaricQuant via CLI with a config specifying isoMethod TMTpro, 20 ppm PPMTolerance/MS1PPMTolerance/MS2PPMTolerance, scanLevel set per acquisition (MS2 or RTS-MS3), and topXNum=10, producing isofrag.csv, isolab.csv, isopep.csv and isopep_extra.csv. Aggregate peptide reporter intensities to protein level, then filter peptides using the MS2 PeptideTopXIntensityFromTotalScore QC score at a cutoff of 0.15 (topX=10, derived from the 25th percentile) before protein quantification. Finally run the thermal-shift significance analysis on unfiltered and filtered quantifications and compare the counts of significant staurosporine targets.

## Related tools

- **IsobaricQuant** (Java-based quantification, visualization, and QC-score filtering of TMTpro-labeled PISA peptides from mzML plus hits-file input; run via CLI to generate isofrag/isolab/isopep CSVs) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (Converts PRIDE PXD014369 raw instrument files to mzML for IsobaricQuant input) — https://isobaricquant.readthedocs.io/en/latest/
- **Comet** (Search engine whose output is converted into the IsobaricQuant hits file format)
- **Percolator** (Validates/weights PSMs before conversion to the IsobaricQuant hits CSV)
- **Java 17** (Required runtime to execute the IsobaricQuant jar) — https://github.com/Villen-Lab/isobaricquant

## Examples

```
java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o output
```

## Evaluation signals

- The four IsobaricQuant output files (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) exist and parse correctly
- MS2-level protein quantification depth is 5,380 proteins and RTS-MS3 depth is 4,033 proteins (within reported tolerances)
- After filtering with PeptideTopXIntensityFromTotalScore cutoff 0.15 (topX=10), filtered protein depth reaches 4,713
- PISA significance analysis reproduces staurosporine target counts: 1 uM increases 4 to 7 and 20 uM increases 17 to 22
- CAMKD2 accuracy and AAPK1 precision examples are confirmed in the reprocessed results

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- ms1PrecSignal is a limited predictor of reporter ion interference because a majority of scans show 100% even when quantitative accuracy is reduced
- Stringent filtering can lead to substantial loss of peptide observations, potentially exacerbating missing values across multiple labeled sample sets
- The PISA RTS-MS3 instrument method acquired a maximum of 2 PSMs per protein, limiting filtering possibilities; higher PSMs per protein are recommended for RTS-MS3

## Evidence
