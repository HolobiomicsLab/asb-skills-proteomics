---
name: sps-ms3-qc-score-filtering
description: Use when quantifying TMT/iTRAQ-labeled peptides with IsobaricQuant (MS2
  or SPS-MS3 mzML) and co-isolation interference must be filtered by QC scores before
  protein-level aggregation or differential testing.
when_to_use_negative:
- The PISA RTS-MS3 acquisition method was set to a maximum of 2 PSMs per protein,
  significantly limiting the possibility of filtering peptides—filtering is not useful
  when so few PSMs per protein exist
- Data are label-free or already aggregated to protein level (no isobaric reporter-ion
  scans and no per-scan QC scores to filter on)
- A fixed published cutoff is applied blindly—optimal score and cutoff are dataset-dependent
  and must be re-optimized per dataset
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: IsobaricQuant
  role: Java-based tool that quantifies, visualizes, and calculates QC scores (e.g.,
    peptideTopXIntensityFromTotalScore) used to filter interfered isobarically-labeled
    scans before protein aggregation
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar via GUI or CLI
- name: msconvert
  role: ProteoWizard tool used to convert Thermo raw files to mzML with vendor peak
    picking before IsobaricQuant quantification
- name: Comet
  role: Search engine whose PSM output is converted into the comma-separated hits
    file consumed by IsobaricQuant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:2cc181a6784ad5af6d8c4dd569a0dab4634a2a2b536cfecdc22be3915326c40e
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# sps-ms3-qc-score-filtering

## Summary

Filters interfered peptide quantifications in isobarically-labeled (TMT/iTRAQ) SPS-MS3 data using IsobaricQuant QC scores—specifically peptideTopXIntensityFromTotalScore with topX=10 and a dataset-derived cutoff (recommended 0.15, 25th percentile as starting point)—prior to protein-level aggregation, so filtering improves accuracy without altering protein inference.

## When to use

Use when quantifying TMT/iTRAQ-labeled peptides with IsobaricQuant (MS2 or SPS-MS3 mzML) and co-isolation interference must be filtered by QC scores before protein-level aggregation or differential testing.

## When NOT to use

- The PISA RTS-MS3 acquisition method was set to a maximum of 2 PSMs per protein, significantly limiting the possibility of filtering peptides—filtering is not useful when so few PSMs per protein exist
- Data are label-free or already aggregated to protein level (no isobaric reporter-ion scans and no per-scan QC scores to filter on)
- A fixed published cutoff is applied blindly—optimal score and cutoff are dataset-dependent and must be re-optimized per dataset

## Inputs

- mzML files converted from Thermo raw files (e.g., via msconvert with vendor peak picking)
- Comma-separated hits file from a search engine (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z; e.g., converted Comet output)
- IsobaricQuant text configuration file (e.g., TMT6plex, scanLevel MS3, PPMTolerance 10, topXNum 10, neutralLosses 63.98,18.01,17.03)
- Optional reporter ion isotopic impurity correction CSV

## Outputs

- IsobaricQuant output CSVs: isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv including QC scores per scan
- Filtered peptide table (rows passing the peptideTopXIntensityFromTotalScore cutoff)
- Protein-level quantification matrix (e.g., >=2 unique peptides per protein; 4,303 proteins pre-filtering in the yeast TMT6plex example)
- Unfiltered vs filtered differential significance comparison (e.g., 45 proteins gaining, 10 losing significance)

## How to apply

Run IsobaricQuant on the mzML files and a search-engine hits CSV (e.g., from Comet) to produce isolab.csv, isopep.csv, isopep_extra.csv, and isofrag.csv with per-scan QC scores. For SPS-MS3 data, compute the 25th percentile of the peptideTopXIntensityFromTotalScore score as a starting cutoff (recommended value 0.15), setting topX=10 to match the 10 SPS-MS3 selection notches; for MS2 data use msnTotalSignal with a recommended cutoff of 1e6. Filter individual peptide/scan rows by the chosen cutoff BEFORE aggregating to proteins (requiring, e.g., at least two unique peptides per protein), because applying filtering directly prior to protein-level quantification assures protein inference is not altered. Re-run protein aggregation and differential testing on both unfiltered and filtered quantifications and tabulate how significance calls shift, confirming the protein count is preserved. Validate by checking that pre-filtering protein counts match (e.g., 4,303 proteins in the yeast glucose-vs-pyruvate PXD014546 analysis) and that per-protein direction changes (e.g., Mdm36 gaining, Gip4 losing significance) are reproduced.

## Related tools

- **IsobaricQuant** (Java-based tool that quantifies, visualizes, and calculates QC scores (e.g., peptideTopXIntensityFromTotalScore) used to filter interfered isobarically-labeled scans before protein aggregation) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Runtime required to execute the IsobaricQuant jar via GUI or CLI)
- **msconvert** (ProteoWizard tool used to convert Thermo raw files to mzML with vendor peak picking before IsobaricQuant quantification)
- **Comet** (Search engine whose PSM output is converted into the comma-separated hits file consumed by IsobaricQuant)

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf yeast.mzML -h comet_hits.csv -o out/
```

## Evaluation signals

- Pre-filtering protein-level aggregation yields the expected protein count (4,303 proteins in the yeast glucose-vs-pyruvate PXD014546 reproduction), confirming protein inference is not altered by filtering
- Cutoff choice traces to the 25th percentile of the peptideTopXIntensityFromTotalScore distribution (recommended 0.15) with topX=10 equal to the number of SPS-MS3 selection notches
- All four expected IsobaricQuant output files (isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv) are produced in the output folder
- Differential significance comparison reproduces direction-of-change calls: 45 proteins gain and 10 lose significance after filtering, with Mdm36 gaining and Gip4 losing
- Protein CVs decrease and inaccurate peptide quantifications are removed relative to the unfiltered quantification

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- Filtering out interfered quantifications improves accuracy and precision but at the cost of significantly decreased peptide observations, potentially exacerbating missing values across multiple labeled sample sets
- ms1PrecSignal is a limited predictor of reporter ion interference—a majority of scans show 100% even when quantitative accuracy is reduced
- The identity of co-isolated peptides and their reporter ion release efficiency is unknown, preventing ratio decompression
- Stringent filtering is not beneficial when few PSMs per protein are available (e.g., the PISA RTS-MS3 method limited to 2 PSMs per protein)

## Evidence
