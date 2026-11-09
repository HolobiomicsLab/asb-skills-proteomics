---
name: scan-group-matching-across-quantification-tools
description: Use when comparing reporter ion intensities produced by two or more isobaric
  quantification tools (e.g. IsobaricQuant vs FragPipe vs MaxQuant) on the same TMT/iTRAQ
  mzML files to verify quantification equivalence.
when_to_use_negative:
- When the goal is filtering interfered quantifications rather than cross-tool benchmarking
  — use IsobaricQuant QC score cutoffs instead
- When raw files are not shared across tools — correlations on non-overlapping scan
  groups are not meaningful
- When labels are not iTRAQ/TMT/TMTpro isobaric tags, since reporter ion quantification
  does not apply
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IsobaricQuant
  role: Primary quantification tool being benchmarked; extracts reporter ion intensities
    from mzML using a hits file and TMTpro config (20 ppm tolerance, scanLevel MS2)
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search engine whose output is converted to the comma-separated hits file format
    required as IsobaricQuant input
- name: msconvert
  role: Converts PXD020815 instrument raw files to mzML with peak picking (peakPicking
    true 1-), --32, --zlib, and zeroSamples removeExtra filters before any tool quantifies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:746ecf535cf4359d3bb281bc96e3d2896ce0e76d2222686702c62996406d9f38
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - msconvert
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Scan-group matching across quantification tools

## Summary

Benchmark reporter ion quantification by joining log10-transformed reporter intensities from IsobaricQuant, FragPipe, and MaxQuant on shared scan groups/peptide identifications and computing pairwise Pearson correlations, as done for the HYpro16 TMTpro 16-plex benchmark (PXD020815).

## When to use

Use when comparing reporter ion intensities produced by two or more isobaric quantification tools (e.g. IsobaricQuant vs FragPipe vs MaxQuant) on the same TMT/iTRAQ mzML files to verify quantification equivalence.

## When NOT to use

- When the goal is filtering interfered quantifications rather than cross-tool benchmarking — use IsobaricQuant QC score cutoffs instead
- When raw files are not shared across tools — correlations on non-overlapping scan groups are not meaningful
- When labels are not iTRAQ/TMT/TMTpro isobaric tags, since reporter ion quantification does not apply

## Inputs

- mzML files converted from PXD020815 instrument raw files (msconvert)
- IsobaricQuant outputs: isolab.csv, isopep.csv, isopep_extra.csv
- FragPipe TMT16 (MS2) and TMT16-MS3 workflow reporter ion outputs
- MaxQuant MS2 reporter ion quantification outputs
- Comet/Mokapot PSM hits file for IsobaricQuant (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)

## Outputs

- quant_comparison_summary.csv with scan-group counts, fully-quantified fractions, and pairwise Pearson correlation coefficients
- Log10-transformed reporter ion intensity join table across shared scan groups/peptides

## How to apply

Run each quantification tool on the same converted mzML files: IsobaricQuant via its CLI (java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o output) with a TMTpro isoMethod config, 20 ppm reporter-ion tolerance, and scanLevel MS2, producing isolab.csv/isopep.csv/isopep_extra.csv; FragPipe with the TMT16 (MS2) and TMT16-MS3 workflows; and MaxQuant with MS2 reporter ion quantification. For each tool, count quantified scan groups and compute the fraction of shared yeast scan groups in which all 16 reporter channels were quantified — this fraction should reach 100% for MS2-level quantification. Then join reporter ion intensities across tools on shared scan groups/peptide identifications, log10-transform the intensities, and compute pairwise Pearson correlations between tools. In the HYpro16 benchmark, log10 reporter ion intensities were perfectly correlated across IsobaricQuant, FragPipe, and MaxQuant, confirming IsobaricQuant quantification is on par with state-of-the-art tools.

## Related tools

- **IsobaricQuant** (Primary quantification tool being benchmarked; extracts reporter ion intensities from mzML using a hits file and TMTpro config (20 ppm tolerance, scanLevel MS2)) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search engine whose output is converted to the comma-separated hits file format required as IsobaricQuant input)
- **msconvert** (Converts PXD020815 instrument raw files to mzML with peak picking (peakPicking true 1-), --32, --zlib, and zeroSamples removeExtra filters before any tool quantifies)

## Examples

```
java -jar IsobaricQuant.jar -c tmtpro_ms2.cfg -mzf PXD020815.mzML -h hits.csv -o output
```

## Evaluation signals

- Pairwise Pearson correlations of log10 reporter intensities between IsobaricQuant, FragPipe, and MaxQuant are essentially perfect (r ≈ 1.0)
- Fraction of shared yeast scan groups with all 16 reporter channels quantified reaches 100% for MS2-level quantification
- Scan-group counts per tool are reported and comparable across MS2, SPS-MS3, and RTS-MS3 acquisition methods
- Join across tools is performed on shared scan groups/peptide identifications only, so unmatched groups are excluded before correlation
- IsobaricQuant run completes on Java 17 and emits isolab.csv, isopep.csv, and isopep_extra.csv

## Limitations

- Perfect correlation confirms concordance of extracted intensities but the paper's benchmark does not address co-isolation identity — 'the identity of co-isolated peptides and their efficiency to release reporter ions is unknown', preventing ratio decompression
- Comparison requires reprocessing the same mzML files with each tool, including tool-specific search/validation steps (Comet + Mokapot for IsobaricQuant)
- Cutoffs and optimal QC scores are dataset-dependent, so benchmark conclusions for HYpro16 may not transfer to other datasets

## Evidence

- [other] Join reporter ion intensities across tools on shared scan groups/peptide identifications and compute Pearson correlations of log10-transformed reporter intensities pairwise between IsobaricQuant, FragPipe, and MaxQuant.: [evidence span withheld — anchor status: not_found]
- [results] Reprocessing the HYpro16 benchmark showed that log 10 reporter ion intensities extracted by IsobaricQuant, FragPipe, and MaxQuant were perfectly correlated, confirming IsobaricQuant quantification is on par with state-of-the-art tools.: [evidence span withheld — anchor status: not_found]
