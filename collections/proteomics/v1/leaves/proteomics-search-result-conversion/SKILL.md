---
name: proteomics-search-result-conversion
description: Use when you have PSMs from a search algorithm such as Comet on mzML-converted
  raw files and need to feed them into IsobaricQuant for MS2/MS3 reporter ion quantification
  and QC-score filtering.
when_to_use_negative:
- Input is already an IsobaricQuant hits file or IsobaricQuant output CSV (isopep.csv/isofrag.csv)
  — no re-conversion is needed
- The search engine output is from a tool whose fields cannot be mapped to Search
  ID, Peptide ID, Sequence, Reference, Charge, Start Scan, and M/Z without unsupported
  assumptions
- The spectra are label-free (not iTRAQ/TMT/TMTpro labeled) — IsobaricQuant quantifies
  isobarically-labeled peptides only
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Target tool that consumes the converted hits file plus mzML and a config file
    via CLI to quantify and score isobarically-labeled peptides
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search algorithm whose output file must be converted into the IsobaricQuant
    hits format
- name: msconvert
  role: ProteoWizard tool used upstream to convert instrument raw files into mzML
    (with peak picking and zero-sample removal filters) before searching
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar in CLI mode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:ccfd32385cffd5c4538bd39fd7416dc9f224affb3d6e41ba83e860b08e90dfd3
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# proteomics-search-result-conversion

## Summary

Convert a peptide search engine output (e.g., Comet results on mzML files converted with msconvert) into the comma-separated hits file required by IsobaricQuant, with the columns Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, and M/Z, so that isobarically-labeled (TMT/iTRAQ) peptides can be quantified, visualized, and QC-filtered.

## When to use

Use when you have PSMs from a search algorithm such as Comet on mzML-converted raw files and need to feed them into IsobaricQuant for MS2/MS3 reporter ion quantification and QC-score filtering.

## When NOT to use

- Input is already an IsobaricQuant hits file or IsobaricQuant output CSV (isopep.csv/isofrag.csv) — no re-conversion is needed
- The search engine output is from a tool whose fields cannot be mapped to Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, and M/Z without unsupported assumptions
- The spectra are label-free (not iTRAQ/TMT/TMTpro labeled) — IsobaricQuant quantifies isobarically-labeled peptides only

## Inputs

- Thermo instrument raw files (e.g., from a PXD dataset)
- mzML files generated with msconvert (profile mode, vendor centroiding/peak picking)
- Search engine output (e.g., Comet PSM results) on the mzML files
- IsobaricQuant text configuration file (e.g., TMT6plex, scanLevel MS3, PPMTolerance 10, topXNum 10, neutralLosses 63.98,18.01,17.03)
- Optional: reporter ion isotopic distribution CSV for isotopic impurity correction

## Outputs

- IsobaricQuant comma-separated hits file with columns Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z
- IsobaricQuant output tables: isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv
- Downstream peptide-level quantifications with QC scores (e.g., peptideTopXIntensityFromTotalScore)

## How to apply

First convert Thermo raw files to mzML with ProteoWizard msconvert (e.g., profile mode with vendor peak picking, --mzML --32 --zlib, 'peakPicking true 1-' and 'zeroSamples removeExtra' filters per the README). Search the mzML files with a supported search algorithm such as Comet, then transform the search output into IsobaricQuant's comma-separated hits file containing the required fields: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, and M/Z. IsobaricQuant additionally requires a user-supplied text configuration file (e.g., TMT6plex, scanLevel MS3, PPMTolerance 10) and optionally a reporter ion isotopic impurity correction CSV. Running 'java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o out' (with Java 17 installed) then produces isolab.csv, isopep.csv, isopep_extra.csv, and isofrag.csv, which are the quantification tables used for downstream peptide- and protein-level aggregation and QC filtering.

## Related tools

- **IsobaricQuant** (Target tool that consumes the converted hits file plus mzML and a config file via CLI to quantify and score isobarically-labeled peptides) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search algorithm whose output file must be converted into the IsobaricQuant hits format)
- **msconvert** (ProteoWizard tool used upstream to convert instrument raw files into mzML (with peak picking and zero-sample removal filters) before searching)
- **Java 17** (Runtime required to execute the IsobaricQuant jar in CLI mode)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf yeast.mzML -h comet_hits.csv -o out
```

## Evaluation signals

- The hits file contains exactly the required columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z
- Running IsobaricQuant CLI completes without errors and emits the four expected output files: isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv
- Start Scan values in the hits file correctly match the scan numbers present in the mzML files so IsobaricQuant can locate the MS2/MS3 spectra
- Config parameters (e.g., TMT6plex, scanLevel MS3, PPMTolerance 10) match the acquisition of the dataset being converted
- Downstream protein-level aggregation on the resulting quantifications reproduces expected protein counts (e.g., 4,303 proteins pre-filtering in the yeast PXD014546 glucose-vs-pyruvate analysis)

## Limitations

- Java 17 is required; older Java versions will not run the IsobaricQuant jar
- Only search engines with output convertible to the required hits schema (e.g., Comet) are supported out of the box
- Cutoff values for downstream QC filtering are dataset-dependent and must be re-optimized per dataset (25th percentile as a starting point)

## Evidence

- [full_text] Search the mzML files with Comet and convert the search output into the IsobaricQuant comma-separated hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z).: [evidence span withheld — anchor status: not_found]
