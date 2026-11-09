---
name: psm-hit-file-parsing
description: Use when a search-engine PSM output (e.g. Comet) must be converted into
  the hits CSV consumed by IsobaricQuant, linking each peptide identification to a
  scan number and precursor M/Z in the mzML file.
when_to_use_negative:
- Input is raw instrument data not yet searched — a PSM hits file from a search engine
  such as Comet is a prerequisite
- 'IsobaricQuant tag scope is exceeded: it supports iTRAQ up to 8-plex, TMT up to
  11-plex and TMTpro up to 16-plex mzML files only'
- The goal is database searching itself — this skill only consumes existing search
  results, it does not perform peptide-spectrum matching
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IsobaricQuant
  role: Consumes the parsed PSM hits CSV together with the mzML and config file to
    compute QC scores and reporter ion quantification
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Source search engine whose output must be converted into the IsobaricQuant
    hits file format
- name: msconvert
  role: Converts instrument raw files to the mzML spectra file that PSM Start Scan
    numbers must map onto
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:5cfce470079d42e9d84fcb54e6b3acfd65b1d97db7cbc7c7102231d209f943f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - msconvert
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# psm-hit-file-parsing

## Summary

Parses and prepares peptide spectral match (PSM) hit files from search engines such as Comet into the comma-separated format IsobaricQuant requires (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) so that QC scores and reporter ion quantification can be computed per scan.

## When to use

Use when a search-engine PSM output (e.g. Comet) must be converted into the hits CSV consumed by IsobaricQuant, linking each peptide identification to a scan number and precursor M/Z in the mzML file.

## When NOT to use

- Input is raw instrument data not yet searched — a PSM hits file from a search engine such as Comet is a prerequisite
- IsobaricQuant tag scope is exceeded: it supports iTRAQ up to 8-plex, TMT up to 11-plex and TMTpro up to 16-plex mzML files only
- The goal is database searching itself — this skill only consumes existing search results, it does not perform peptide-spectrum matching

## Inputs

- Search engine PSM output file (e.g. Comet results) to be converted
- CSV hits file with columns Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z
- mzML file of the MS run (from msconvert with --mzML, --simAsSpectra, --32, --zlib, peakPicking true 1-, zeroSamples removeExtra)
- IsobaricQuant text configuration file (scanLevel, PPMTolerance 20, MS1PPMTolerance 20, MS2PPMTolerance 20, topXNum 10)

## Outputs

- Four output CSVs: isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv with QC score columns (ms1PrecSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore, msnTotalSignal)

## How to apply

Convert the search engine output — IsobaricQuant requires 'a conversion of the output file of a search algorithm such as Comet' — into a CSV hits file with the documented columns: Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z. Supply this hits file to IsobaricQuant alongside the mzML file (produced via msconvert with peakPicking and zeroSamples filters) and a config file setting scanLevel, PPMTolerance 20, MS1PPMTolerance 20, MS2PPMTolerance 20, and topXNum 10. The Start Scan column must map PSMs onto scan numbers in the mzML so that MS1 precursor purity, MS2 peptide purity, and MS3 SPS purity scores are computed for the right spectra. The parsed hits feed computation of peptideIntensityScore (totalFragmIonsIntensity/totalIntensity) and peptideTopXIntensityFromTotalScore (topX=10) by matching b/y fragment ions with neutral losses 63.98, 18.01, 17.03. Correct parsing is verified when every score column appears in isopep_extra.csv with values in valid ranges (e.g. peptideIntensityScore in [0,1]) and isolab.csv label intensities are non-negative.

## Related tools

- **IsobaricQuant** (Consumes the parsed PSM hits CSV together with the mzML and config file to compute QC scores and reporter ion quantification) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Source search engine whose output must be converted into the IsobaricQuant hits file format)
- **msconvert** (Converts instrument raw files to the mzML spectra file that PSM Start Scan numbers must map onto)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf HYpro16.mzML -h psm_hits.csv -o out/
```

## Evaluation signals

- Each documented hits column (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z) is present and populated in the parsed input
- Every PSM Start Scan maps to an existing scan number in the mzML file so QC scores attach to real spectra
- All four output CSVs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) are written and isopep_extra.csv contains each score column defined in Table 1
- Score values fall in valid ranges: peptideIntensityScore and peptideTopXIntensityFromTotalScore in [0,1], ms1PrecSignal percentages up to 100%
- Label intensities in isolab.csv are non-negative with matched peaks recorded within the +/- PPMTolerance (20 ppm) window

## Limitations

- The search-engine output requires conversion before use; raw Comet output is not directly consumable
- Only iTRAQ (up to 8-plex), TMT (up to 11-plex) and TMTpro (up to 16-plex) mzML files are supported
- No accession or data availability identifier is given for the PISA staurosporine dataset used to exercise the pipeline, complicating reproducibility of the parsing workflow end-to-end

## Evidence
