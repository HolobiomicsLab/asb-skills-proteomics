---
name: log10-reporter-intensity-correlation-analysis
description: Use when validating that a new reporter-ion quantification tool produces
  intensities consistent with tools, by reprocessing the same mzML files with multiple
  tools and correlating their log10 reporter intensities on shared scan groups.
when_to_use_negative:
- The input data are not isobarically labeled (iTRAQ/TMT/TMTpro) — reporter-ion quantification
  tools are not applicable to label-free data.
- The goal is to assess quantitative accuracy of individual scans (ratio compression)
  rather than cross-tool intensity agreement; use IsobaricQuant QC-score filtering
  instead.
- Plex sizes exceed tool support (IsobaricQuant supports up to iTRAQ 8-plex, TMT 11-plex,
  TMTpro 16-plex).
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Primary quantification tool under benchmark; extracts reporter ion intensities
    from mzML files via CLI and produces isolab.csv/isopep.csv/isopep_extra.csv
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search engine producing PSM output that is converted into the IsobaricQuant
    hits file format
  repo: https://uwpr.github.io/Comet/
- name: msconvert
  role: ProteoWizard tool used to convert instrument raw files to mzML before quantification
  repo: http://proteowizard.sourceforge.net/tools/filters.html
- name: FragPipe
  role: Reference tool used with TMT16 (MS2) and TMT16-MS3 workflows to quantify the
    same mzML files for correlation comparison
- name: MaxQuant
  role: Reference tool used with MS2 reporter ion quantification to quantify the same
    mzML files for correlation comparison
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar via the command line
  repo: https://github.com/Villen-Lab/isobaricquant
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:0daee6a46f477f699e7eb411816ddd33df476e4b76fb2684a15f7622eaec3436
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - FragPipe
  - IsobaricQuant
  - Java
  - Java 17
  - MaxQuant
  - msconvert
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# log10-reporter-intensity-correlation-analysis

## Summary

Benchmark a novel isobaric quantification tool against established software (FragPipe, MaxQuant) by joining reporter ion intensities on shared scan groups and computing pairwise Pearson correlations of log10-transformed reporter intensities. Applied here to validate IsobaricQuant on the HYpro16 TMTpro 16-plex dataset across MS2, SPS-MS3, and RTS-MS3 acquisition methods.

## When to use

Use when validating that a new reporter-ion quantification tool produces intensities consistent with state-of-the-art tools, by reprocessing the same mzML files with multiple tools and correlating their log10 reporter intensities on shared scan groups.

## When NOT to use

- The input data are not isobarically labeled (iTRAQ/TMT/TMTpro) — reporter-ion quantification tools are not applicable to label-free data.
- The goal is to assess quantitative accuracy of individual scans (ratio compression) rather than cross-tool intensity agreement; use IsobaricQuant QC-score filtering instead.
- Plex sizes exceed tool support (IsobaricQuant supports up to iTRAQ 8-plex, TMT 11-plex, TMTpro 16-plex).

## Inputs

- PXD020815 instrument raw files (HYpro16 TMTpro 16-plex, PRIDE)
- mzML files converted from raw with msconvert
- PSM hits CSV file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) from Comet/Mokapot
- IsobaricQuant TMTpro isoMethod configuration file (20 ppm reporter-ion tolerance, scanLevel MS2)
- FragPipe TMT16 (MS2) and TMT16-MS3 workflow outputs
- MaxQuant MS2 reporter ion quantification output

## Outputs

- IsobaricQuant outputs isolab.csv, isopep.csv, isopep_extra.csv
- quant_comparison_summary.csv with scan-group counts, fully-quantified fractions, and pairwise Pearson correlation coefficients of log10 reporter intensities

## How to apply

Download the PXD020815 instrument raw files from PRIDE and convert them to mzML with ProteoWizard's msconvert (e.g., via the chambm/pwiz Docker image with --mzML, peakPicking and zeroSamples filters). Search the spectra with Comet, validate PSMs with Mokapot, and convert results into the comma-separated hits file format required by IsobaricQuant (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z). Run IsobaricQuant via its CLI on Java 17 with a TMTpro isoMethod configuration, 20 ppm reporter-ion tolerance, and scanLevel MS2, generating isolab.csv/isopep.csv/isopep_extra.csv; quantify the same mzML files with FragPipe (TMT16 MS2 and TMT16-MS3 workflows) and MaxQuant (MS2 reporter ion quantification). For each tool, count quantified scan groups and compute the fraction of shared yeast scan groups in which all 16 reporter channels were quantified (expecting 100% at MS2 level). Join reporter intensities across tools on shared scan groups/peptide identifications, log10-transform them, and compute pairwise Pearson correlations; write scan-group counts, fully-quantified fractions, and correlation coefficients to quant_comparison_summary.csv.

## Related tools

- **IsobaricQuant** (Primary quantification tool under benchmark; extracts reporter ion intensities from mzML files via CLI and produces isolab.csv/isopep.csv/isopep_extra.csv) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search engine producing PSM output that is converted into the IsobaricQuant hits file format) — https://uwpr.github.io/Comet/
- **msconvert** (ProteoWizard tool used to convert instrument raw files to mzML before quantification) — http://proteowizard.sourceforge.net/tools/filters.html
- **FragPipe** (Reference tool used with TMT16 (MS2) and TMT16-MS3 workflows to quantify the same mzML files for correlation comparison)
- **MaxQuant** (Reference tool used with MS2 reporter ion quantification to quantify the same mzML files for correlation comparison)
- **Java 17** (Runtime required to execute the IsobaricQuant jar via the command line) — https://github.com/Villen-Lab/isobaricquant

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf HYpro16.mzML -h hits.csv -o output
```

## Evaluation signals

- Pairwise Pearson correlations of log10-transformed reporter intensities between IsobaricQuant, FragPipe, and MaxQuant should be perfect (1.0) on shared scan groups, confirming quantification is on par with state-of-the-art tools.
- The fraction of shared yeast scan groups in which all 16 reporter channels were quantified must reach 100% for MS2-level quantification.
- IsobaricQuant must run successfully on Java 17 and emit the expected isolab.csv/isopep.csv/isopep_extra.csv output files.
- quant_comparison_summary.csv must contain scan-group counts, fully-quantified fractions, and correlation coefficients for all three tools.
- Reporter intensities must join cleanly across tools on shared scan groups/peptide identifications.

## Limitations

- IsobaricQuant supports iTRAQ up to 8-plex, TMT up to 11-plex, and TMTpro up to 16-plex only.
- Optimal QC scores and cutoffs are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization, and filtering trades improved accuracy for decreased peptide observations.
- The benchmark repository (isobaricquant_benchmark) is at first submission stage; users must download all associated raw/mzML files, IsobaricQuant config files, and search engine hit files pointed to within the scripts before running them.

## Evidence

- [readme] Java 17 requirement: [evidence span withheld — anchor status: external_doc]
