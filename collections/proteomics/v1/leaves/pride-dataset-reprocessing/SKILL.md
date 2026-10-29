---
name: pride-dataset-reprocessing
description: Use when a raw dataset from a PRIDE accession (e.g. PXD014369) must be
  reprocessed from scratch to reproduce reported results — converting raw files, searching
  spectra, quantifying isobaric labels, and validating against published output counts.
when_to_use_negative:
- Input data are already mzML or a protein-level feature table and the goal is only
  downstream analysis — skip the raw-file conversion and search steps
- The dataset is label-free or uses non-isobaric labeling (iTRAQ/TMT/TMTpro are the
  supported tags up to 16-plex TMTpro)
- The PRIDE accession does not provide raw instrument files, so the reprocessing workflow
  cannot be started from raw data
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: msconvert
  role: Converts the raw instrument file to mzML with peakPicking and zeroSamples
    filters before IsobaricQuant processing
- name: Comet
  role: Search engine whose output file is converted to the IsobaricQuant hits CSV
    format
- name: Percolator
  role: Validates and weights PSMs from the Comet search prior to hits-file conversion
- name: IsobaricQuant
  role: CLI-mode quantification and QC-score filtering of isobarically-labeled peptides
    from the mzML and hits files
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java
  role: Runtime required to run the IsobaricQuant jar (Java 17)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:66121e0a7d7513815026647dbc703eab52fac794fca4ce7cbf2d576637245c8d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - Percolator
  - msconvert
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pride-dataset-reprocessing

## Summary

End-to-end reprocessing of a public PRIDE proteomics dataset (e.g. the staurosporine PISA dataset PXD014369) from raw instrument files through mzML conversion, database search, peptide validation, isobaric quantification, and downstream significance analysis, reproducing the paper's reported protein-depth and target-count metrics.

## When to use

Use when a raw dataset from a PRIDE accession (e.g. PXD014369) must be reprocessed from scratch to reproduce reported results — converting raw files, searching spectra, quantifying isobaric labels, and validating against published output counts.

## When NOT to use

- Input data are already mzML or a protein-level feature table and the goal is only downstream analysis — skip the raw-file conversion and search steps
- The dataset is label-free or uses non-isobaric labeling (iTRAQ/TMT/TMTpro are the supported tags up to 16-plex TMTpro)
- The PRIDE accession does not provide raw instrument files, so the reprocessing workflow cannot be started from raw data

## Inputs

- Raw instrument files from a PRIDE accession (e.g. PXD014369, 1 and 20 μM staurosporine PISA runs)
- IsobaricQuant text configuration file (isoMethod TMTpro, PPMTolerance 20 ppm, scanLevel, topXNum 10)
- Comet search output converted to IsobaricQuant hits CSV (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- Optionally, reporter ion isotopic impurity correction CSV

## Outputs

- mzML files from msconvert
- PSM hits file after Comet/Percolator validation
- IsobaricQuant CSV outputs: isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv
- Protein-level quantification table and counts (MS2: 5,380 proteins; RTS-MS3: 4,033; filtered: 4,713)
- PISA thermal-shift significance results (significant staurosporine targets per dose)

## How to apply

Download the raw instrument files for the relevant experiment (e.g. PISA staurosporine 1 and 20 μM, PRIDE PXD014369) and convert them to open mzML format with msconvert using the IsobaricQuant-recommended filters (peakPicking true 1-, zeroSamples removeExtra). Search the spectra with Comet, validate/weight PSMs with Percolator, and convert the search output to the comma-separated hits file format required by IsobaricQuant (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z). Run IsobaricQuant in CLI mode (java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o output) using the TMTpro isoMethod, 20 ppm PPMTolerance/MS1PPMTolerance/MS2PPMTolerance, scanLevel set per acquisition (MS2 or RTS-MS3), and topXNum = 10, which generates isofrag.csv, isolab.csv, isopep.csv and isopep_extra.csv. Aggregate peptide reporter intensities to the protein level, optionally applying QC-score filtering (e.g. MS2 PeptideTopXIntensityFromTotalScore at the 25th-percentile-derived cutoff 0.15 with topX = 10) before protein quantification, then run the downstream analysis (e.g. PISA thermal-shift significance) and compare counts against reported targets (MS2 depth 5,380, RTS-MS3 4,033, filtered 4,713; staurosporine targets 1 μM: 4→7, 20 μM: 17→22).

## Related tools

- **msconvert** (Converts the raw instrument file to mzML with peakPicking and zeroSamples filters before IsobaricQuant processing)
- **Comet** (Search engine whose output file is converted to the IsobaricQuant hits CSV format)
- **Percolator** (Validates and weights PSMs from the Comet search prior to hits-file conversion)
- **IsobaricQuant** (CLI-mode quantification and QC-score filtering of isobarically-labeled peptides from the mzML and hits files) — https://github.com/Villen-Lab/isobaricquant
- **Java** (Runtime required to run the IsobaricQuant jar (Java 17))

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf PXD014369_mzML -h percolator_hits.csv -o out
```

## Evaluation signals

- All four IsobaricQuant CSV outputs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) exist and parse correctly
- MS2-level protein quantification count matches ~5,380 proteins
- RTS-MS3-level protein quantification count matches ~4,033 proteins
- Filtered MS2 protein depth (PeptideTopXIntensityFromTotalScore cutoff 0.15, topX 10) matches ~4,713 proteins
- PISA significant-target counts reproduce the reported change from 4 to 7 at 1 μM and 17 to 22 at 20 μM staurosporine, with CAMKD2 accuracy and AAPK1 precision examples confirmed

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a good starting value for cutoff optimization
- Stringent filtering can lead to substantial loss of peptide observations, exacerbating missing values across multiple labeled sample sets, although protein-level depth is largely preserved
- The PISA RTS-MS3 method acquired a maximum of 2 PSMs per protein, which limited filtering possibilities; higher PSMs per protein are recommended for RTS-MS3 acquisition
- ms1PrecSignal is a limited predictor of interference since a majority of scans show 100% even when accuracy is reduced

## Evidence

- [other] Applying upfront scan-level filtering with IsobaricQuant to the MS2 PISA data increased the number of proteins with significant thermal-stability change from 4 to 7 at 1 μM staurosporine and from 17 to 22 at 20 μM.: [evidence span withheld — anchor status: not_found]
- [readme] Recommended msconvert filters documented for IsobaricQuant: [evidence span withheld — anchor status: external_doc]
