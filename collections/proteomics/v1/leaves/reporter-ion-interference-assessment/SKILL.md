---
name: reporter-ion-interference-assessment
description: Use when quantifying isobarically-labeled (TMT, TMTpro, iTRAQ) peptides
  where co-isolation of co-eluting precursors may compress reporter ion ratios, and
  you need to audit or filter scans at MS2 or MS3 level before protein-level quantification.
when_to_use_negative:
- Input is label-free or otherwise non-isobaric data — the QC scores target iTRAQ/TMT/TMTpro
  reporter ions only
- Acquisition is RTS-MS3 with very few PSMs per protein (e.g. maximum of 2), which
  significantly limits the possibility of filtering peptides
- A fixed, dataset-independent cutoff is required — the paper states both optimal
  score and cutoff are dataset-dependent and must be re-optimized
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Computes reporter ion interference QC scores, provides the viewer for auditing
    isolation purity and SPS selection, and supports score-based filtering via CLI
    or GUI
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: Converts vendor raw files to the mzML input that IsobaricQuant requires
- name: Comet
  role: Search engine whose output is converted into the IsobaricQuant comma-separated
    hits file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:8982d6a29eee788bec3cc378aef1e25b33affe15f79f31e9b9a068e9eb6fb9e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - msconvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# reporter-ion-interference-assessment

## Summary

Assess and filter TMT/iTRAQ reporter ion quantifications compromised by co-isolated, co-eluting precursors using IsobaricQuant's QC scores and integrated viewer, so that only scans with reliable reporter ion intensities feed protein-level aggregation.

## When to use

Use when quantifying isobarically-labeled (TMT, TMTpro, iTRAQ) peptides where co-isolation of co-eluting precursors may compress reporter ion ratios, and you need to audit or filter scans at MS2 or MS3 level before protein-level quantification.

## When NOT to use

- Input is label-free or otherwise non-isobaric data — the QC scores target iTRAQ/TMT/TMTpro reporter ions only
- Acquisition is RTS-MS3 with very few PSMs per protein (e.g. maximum of 2), which significantly limits the possibility of filtering peptides
- A fixed, dataset-independent cutoff is required — the paper states both optimal score and cutoff are dataset-dependent and must be re-optimized

## Inputs

- mzML file of the MS run (converted from Thermo raw via msconvert with vendor peak picking)
- CSV hits file of PSMs from a search engine such as Comet (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- User-supplied IsobaricQuant text configuration file (e.g. TMT6plex, scanLevel MS3, PPMTolerance 10, neutralLosses 63.98,18.01,17.03)
- Optional reporter ion isotopic distribution CSV for isotopic impurity correction

## Outputs

- isolab.csv, isopep.csv, isopep_extra.csv and isofrag.csv quantification/QC tables in the output folder
- Per-scan QC scores (peptideTopXIntensityFromTotalScore, peptideIntensityScore, msnTotalSignal, ms1PrecSignal)
- Filtered peptide-to-protein aggregation and pre/post-filtering differential significance comparison

## How to apply

Run IsobaricQuant (Java 17, java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o out) on the mzML file and a search-engine hits CSV to compute QC scores per scan, then inspect isolation purity, reporter ion quantification, and SPS ion selection in the integrated viewer. For SPS-MS3 data, use peptideTopXIntensityFromTotalScore with topX = 10 (equal to the number of SPS-MS3 selection notches) and a recommended cutoff of 0.15; for MS3 generally peptideIntensityScore (cutoff 0.25) also performs well, and for MS2-level data msnTotalSignal with a recommended cutoff of 1e6. Start cutoff optimization at the 25th percentile of the score distribution and tune per dataset, since optimal score and cutoff are dataset-dependent. Apply filtering directly prior to protein-level quantification so protein inference is not altered, then aggregate peptides to proteins (e.g. requiring at least two unique peptides) and re-run differential testing on unfiltered vs filtered quantifications to quantify changed significance outcomes.

## Related tools

- **IsobaricQuant** (Computes reporter ion interference QC scores, provides the viewer for auditing isolation purity and SPS selection, and supports score-based filtering via CLI or GUI) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (Converts vendor raw files to the mzML input that IsobaricQuant requires)
- **Comet** (Search engine whose output is converted into the IsobaricQuant comma-separated hits file)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o output_folder
```

## Evaluation signals

- Check that all four expected output files (isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv) are produced in the output folder
- Confirm the QC score chosen matches the acquisition level: peptideTopXIntensityFromTotalScore (topX = 10, cutoff 0.15) or peptideIntensityScore (0.25) for MS3; msnTotalSignal (1e6) for MS2
- Verify protein inference is unchanged by filtering (same protein-level aggregation logic and unique-peptide requirement applied before and after)
- In the yeast glucose-vs-pyruvate benchmark, pre-filtering protein count should equal 4,303 and filtering should shift 45 proteins toward and 10 away from significance (Mdm36 gains, Gip4 loses)
- Confirm ms1PrecSignal is not used as the primary filter, since most scans show 100% even when quantitative accuracy is reduced

## Limitations

- Optimal QC score and cutoff are dataset-dependent; score distributions differed across the three datasets analyzed
- Filtering improves accuracy and precision but significantly decreases peptide observations, potentially exacerbating missing values across multiple labeled sample sets
- ms1PrecSignal is a limited predictor of interference — a majority of scans show 100% even when accuracy is reduced
- The identity of co-isolated peptides and their reporter ion release efficiency remain unknown, preventing ratio decompression

## Evidence
