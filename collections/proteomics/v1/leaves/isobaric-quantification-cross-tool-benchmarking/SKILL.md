---
name: isobaric-quantification-cross-tool-benchmarking
description: Use when validating that a new or custom reporter-ion quantification
  tool produces intensities consistent with established software (FragPipe, MaxQuant)
  on isobarically-labeled TMT/TMTpro data acquired at MS2 or MS3 level.
when_to_use_negative:
- The goal is filtering inaccurate quantifications rather than benchmarking tool agreement
  — use IsobaricQuant QC scores (msnTotalSignal cutoff 1e6 for MS2; peptideIntensityScore
  0.25 or peptideTopXIntensityFromTotalScore 0.15 for MS3) instead
- The data are label-free or use labels other than iTRAQ (up to 8-plex), TMT (up to
  11-plex), or TMTpro (up to 16-plex), which IsobaricQuant does not support
- Only one quantification tool is available, so no cross-tool correlation benchmark
  is possible
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Primary tool under benchmark; extracts reporter ion intensities from mzML
    files using a hits file and TMTpro isoMethod config via its CLI
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: Converts PRIDE instrument raw files to mzML, the input format required by
    IsobaricQuant
- name: Comet
  role: Search engine whose output is converted into the comma-separated hits file
    format required by IsobaricQuant
- name: Java 17
  role: Runtime dependency required to execute the IsobaricQuant jar
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:742d63d8a0a5af820c261f230293d187a47d8722925a5819bbf8e71622e37153
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# isobaric-quantification-cross-tool-benchmarking

## Summary

Benchmark IsobaricQuant reporter-ion quantification against FragPipe and MaxQuant on the HYpro16 TMTpro 16-plex dataset (PXD020815) across MS2, SPS-MS3, and RTS-MS3 acquisition methods, verifying agreement via Pearson correlations of log10 reporter intensities on shared scan groups.

## When to use

Use when validating that a new or custom reporter-ion quantification tool produces intensities consistent with established software (FragPipe, MaxQuant) on isobarically-labeled TMT/TMTpro data acquired at MS2 or MS3 level.

## When NOT to use

- The goal is filtering inaccurate quantifications rather than benchmarking tool agreement — use IsobaricQuant QC scores (msnTotalSignal cutoff 1e6 for MS2; peptideIntensityScore 0.25 or peptideTopXIntensityFromTotalScore 0.15 for MS3) instead
- The data are label-free or use labels other than iTRAQ (up to 8-plex), TMT (up to 11-plex), or TMTpro (up to 16-plex), which IsobaricQuant does not support
- Only one quantification tool is available, so no cross-tool correlation benchmark is possible

## Inputs

- PXD020815 instrument raw files (HYpro16 TMTpro 16-plex, yeast) from PRIDE
- mzML files converted from raw files with msconvert
- Comet/Mokapot-validated PSM hits file in CSV format (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- IsobaricQuant text configuration file (TMTpro isoMethod, 20 ppm reporter-ion tolerance, scanLevel MS2)

## Outputs

- IsobaricQuant quantification outputs: isolab.csv, isopep.csv, isopep_extra.csv
- FragPipe and MaxQuant reporter-ion intensity tables for the same mzML files
- quant_comparison_summary.csv with scan-group counts, fully-quantified fractions, and pairwise Pearson correlation coefficients of log10 reporter intensities

## How to apply

Download the PXD020815 instrument raw files from PRIDE and convert them to mzML with msconvert (e.g. via the chambm/pwiz Docker image with peakPicking and zeroSamples filters). Search the spectra with Comet, validate PSMs with Mokapot, and convert results into the comma-separated hits file format (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) required by IsobaricQuant. Run IsobaricQuant on Java 17 with a TMTpro isoMethod configuration, 20 ppm reporter-ion tolerance, and scanLevel MS2 to produce isolab.csv/isopep.csv/isopep_extra.csv; quantify the same mzML files with FragPipe (TMT16 MS2 and TMT16-MS3 workflows) and MaxQuant (MS2 reporter ion quantification). Count quantified scan groups per tool, compute the fraction of shared yeast scan groups with all 16 reporter channels quantified (expecting 100% at MS2 level), join intensities across tools on shared scan groups/peptide identifications, and compute pairwise Pearson correlations of log10-transformed reporter intensities, writing results to quant_comparison_summary.csv.

## Related tools

- **IsobaricQuant** (Primary tool under benchmark; extracts reporter ion intensities from mzML files using a hits file and TMTpro isoMethod config via its CLI) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (Converts PRIDE instrument raw files to mzML, the input format required by IsobaricQuant)
- **Comet** (Search engine whose output is converted into the comma-separated hits file format required by IsobaricQuant)
- **Java 17** (Runtime dependency required to execute the IsobaricQuant jar)

## Examples

```
java -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>
```

## Evaluation signals

- Pairwise Pearson correlations of log10 reporter ion intensities between IsobaricQuant, FragPipe, and MaxQuant are ~1 (perfectly correlated) on shared scan groups
- The fraction of shared yeast scan groups with all 16 reporter channels quantified reaches 100% for MS2-level quantification
- IsobaricQuant produces the expected three output files: isolab.csv, isopep.csv, and isopep_extra.csv
- Scan-group counts are tabulated per tool and per acquisition method (MS2, SPS-MS3, RTS-MS3) in quant_comparison_summary.csv
- The 20 ppm reporter-ion tolerance and TMTpro isoMethod settings are correctly specified in the IsobaricQuant configuration

## Limitations

- IsobaricQuant quantifications of labeled peptides suffer reduced accuracy due to co-isolation of co-eluting precursors of similar mass-to-charge; the identity of co-isolated peptides and their reporter ion release efficiency remain unknown, preventing ratio decompression
- Stringent filtering can lead to substantial loss of peptide observations, potentially exacerbating missing values when multiple sets of labeled samples are measured
- Optimal QC score and cutoff are dataset-dependent; the 25th percentile proved a good starting value for cutoff optimization but cutoffs are not universally applicable

## Evidence
