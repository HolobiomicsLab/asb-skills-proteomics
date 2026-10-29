---
name: differential-abundance-significance-testing
description: Use when two isobarically-labeled conditions (e.g., TMT6plex SPS-MS3
  glucose vs pyruvate) must be compared at the protein level, especially to test whether
  QC-score-based scan filtering shifts the set of significant proteins.
when_to_use_negative:
- Input is already an aggregated protein-level feature table with no scan/peptide-level
  QC scores available, so the pre- vs post-filtering significance comparison cannot
  be performed
- Data are not isobarically-labeled (label-free or SILAC) quantifications, where reporter-ion
  QC scores like peptideTopXIntensityFromTotalScore are undefined
- RTS-MS3 acquisition acquired only ~2 PSMs per protein, which significantly limits
  the possibility of filtering peptides and makes the filtering-dependent significance
  comparison uninformative
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Quantifies TMT6plex reporter ions at scan level, computes QC scores used for
    filtering, and outputs the peptide tables aggregated for differential significance
    testing
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search engine whose PSM output is converted into the IsobaricQuant hits CSV
    input
- name: msconvert
  role: Converts Thermo raw files to mzML (profile mode with peak picking) for IsobaricQuant
    input
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar in CLI mode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:17145247ecc9b66f2f6d0c6c37b409264dbb8fac8e2176ea6153832c5a57397c
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# differential-abundance-significance-testing

## Summary

Protein-level differential significance testing between two growth conditions (e.g., yeast glucose vs pyruvate) on isobarically-labeled (TMT6plex SPS-MS3) quantifications, performed both before and after scan-level QC filtering with IsobaricQuant to reveal how filtering changes which proteins are called significant.

## When to use

Use when two isobarically-labeled conditions (e.g., TMT6plex SPS-MS3 glucose vs pyruvate) must be compared at the protein level, especially to test whether QC-score-based scan filtering shifts the set of significant proteins.

## When NOT to use

- Input is already an aggregated protein-level feature table with no scan/peptide-level QC scores available, so the pre- vs post-filtering significance comparison cannot be performed
- Data are not isobarically-labeled (label-free or SILAC) quantifications, where reporter-ion QC scores like peptideTopXIntensityFromTotalScore are undefined
- RTS-MS3 acquisition acquired only ~2 PSMs per protein, which significantly limits the possibility of filtering peptides and makes the filtering-dependent significance comparison uninformative

## Inputs

- Thermo raw files from a TMT6plex SPS-MS3 run (PXD014546, S. cerevisiae grown on glucose vs pyruvate)
- mzML files converted with msconvert (profile mode, vendor centroiding)
- Comet search output converted to the IsobaricQuant comma-separated hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- IsobaricQuant text configuration file (TMT6plex, scanLevel MS3, PPMTolerance 10, topXNum 10, neutralLosses 63.98,18.01,17.03)

## Outputs

- IsobaricQuant output tables isolab.csv, isopep.csv, isopep_extra.csv and isofrag.csv
- Pre-filtering protein-level quantification (4,303 proteins at >=2 unique peptides per protein)
- Filtered protein-level quantification after peptide filtering at the 25th-percentile peptideTopXIntensityFromTotalScore cutoff (0.15)
- Tabulated significance changes: 45 proteins gained and 10 lost significance, with Mdm36 gaining and Gip4 losing

## How to apply

Convert PXD014546 Thermo raw files to mzML with msconvert (profile mode with vendor peak picking), search with Comet, and format hits as the IsobaricQuant CSV (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z). Run IsobaricQuant CLI configured for TMT6plex, scanLevel MS3, PPMTolerance 10, topXNum 10, and neutralLosses 63.98,18.01,17.03, producing isolab.csv, isopep.csv, isopep_extra.csv and isofrag.csv. Aggregate peptides to proteins requiring at least two unique peptides per protein (yielding 4,303 pre-filtering proteins), then repeat aggregation after filtering peptides by the 25th percentile of peptideTopXIntensityFromTotalScore (topX = 10, matching the 10 SPS-MS3 selection notches; recommended cutoff 0.15). Run the glucose-vs-pyruvate differential significance test on both the unfiltered and filtered protein quantifications and tabulate proteins that gain or lose significance. Filtering must be applied directly prior to protein-level quantification so that protein inference is not altered, ensuring significance changes reflect quantification accuracy rather than membership changes.

## Related tools

- **IsobaricQuant** (Quantifies TMT6plex reporter ions at scan level, computes QC scores used for filtering, and outputs the peptide tables aggregated for differential significance testing) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search engine whose PSM output is converted into the IsobaricQuant hits CSV input)
- **msconvert** (Converts Thermo raw files to mzML (profile mode with peak picking) for IsobaricQuant input)
- **Java 17** (Runtime required to execute the IsobaricQuant jar in CLI mode)

## Examples

```
java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o out
```

## Evaluation signals

- Reproduced pre-filtering protein count equals 4,303 when aggregating with a minimum of two unique peptides per protein
- Filtered significance-change counts equal 45 proteins gained and 10 lost significance versus the unfiltered comparison
- Per-protein direction changes reproduce: Mdm36 gains significance and Gip4 loses significance after filtering
- Filtering is applied at the peptide level using the 25th percentile of peptideTopXIntensityFromTotalScore (topX = 10, cutoff 0.15) before protein aggregation, and protein inference is confirmed unchanged between unfiltered and filtered runs
- IsobaricQuant outputs the four expected files (isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv) with the specified config (TMT6plex, scanLevel MS3, PPMTolerance 10, topXNum 10)

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization, so cutoffs must be adjusted per dataset
- Filtering improves quantification accuracy and precision but significantly decreases peptide observations, exacerbating missing values when multiple labeled sets are measured
- Stringent filtering trades sensitivity: fewer peptide observations may reduce statistical power in the differential significance test

## Evidence

- [abstract] QC scores for filtering significance testing: "its QC scores can successfully filter out scans with reduced quantitative accuracy at MS2 and MS3 levels, removing inaccurate peptide quantifications and decreasing protein CVs"
- [discussion] Recommended MS3 cutoffs and topX rationale: "great filtering results using peptideIntensityScore and peptideTopXIntensityFromTotalScore, with recommended cutoffs 0.25 and 0.15, respectively"
- [discussion] Filter placement preserves protein inference: "applying this filtering step directly prior to protein-level quantification assures that protein inference is not impacted or altered in any way"
- [methods] Java runtime requirement: "In order to run ``IsobaricQuant`` Java 17 is needed"
- [readme] CLI invocation syntax: "java -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>"
- [discussion] Cutoff optimization guidance: "the 25th percentile proved to be a good starting value for cutoff optimization"
