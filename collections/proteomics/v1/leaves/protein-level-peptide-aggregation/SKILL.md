---
name: protein-level-peptide-aggregation
description: Use when IsobaricQuant peptide quantification output (isofrag.csv / isopep.csv)
  from TMT or iTRAQ mzML data must be rolled up to protein-level quantities for downstream
  differential significance testing.
when_to_use_negative:
- Input is already a protein-level feature table or has been aggregated with a different
  protein inference step — re-aggregation would change inference
- Proteins quantified by a single peptide only, since the aggregation rule requires
  at least two unique peptides per protein
- Datasets where a max-PSMs-per-protein acquisition (e.g., 2 PSMs per protein in RTS-MS3)
  leaves too few peptides per protein for filtering to act meaningfully
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IsobaricQuant
  role: Quantifies isobarically-labeled peptides and computes QC scores used to filter
    peptides before protein-level aggregation; run via CLI
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java
  role: Runtime (Java 17) required to execute the IsobaricQuant jar that produces
    the peptide quantification inputs
- name: msconvert
  role: Converts Thermo raw files to mzML (with vendor peak picking) that serve as
    IsobaricQuant input upstream of aggregation
- name: Comet
  role: Search engine whose PSM output is converted into the IsobaricQuant hits CSV
    supplying peptide-to-protein references
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:b19c845f6e02b39895dd9c55ae931178201391a841015caca79f7b92b7d25e40
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - Comet
  - IsobaricQuant
  - Java
  - msconvert
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# protein-level-peptide-aggregation

## Summary

Aggregates IsobaricQuant-quantified peptides (e.g., TMT6plex SPS-MS3 reporter ion intensities from isofrag.csv) to protein-level quantification, requiring at least two unique peptides per protein, with QC-score-based peptide filtering applied directly before aggregation so protein inference is preserved.

## When to use

Use when IsobaricQuant peptide quantification output (isofrag.csv / isopep.csv) from TMT or iTRAQ mzML data must be rolled up to protein-level quantities for downstream differential significance testing.

## When NOT to use

- Input is already a protein-level feature table or has been aggregated with a different protein inference step — re-aggregation would change inference
- Proteins quantified by a single peptide only, since the aggregation rule requires at least two unique peptides per protein
- Datasets where a max-PSMs-per-protein acquisition (e.g., 2 PSMs per protein in RTS-MS3) leaves too few peptides per protein for filtering to act meaningfully

## Inputs

- IsobaricQuant output CSVs (isofrag.csv, isopep.csv, isopep_extra.csv, isolab.csv) from TMT/iTRAQ-labeled mzML runs
- Peptide-to-protein mapping (Reference/protein groups from the Comet hits file)
- QC score column (peptideTopXIntensityFromTotalScore) with a 25th-percentile-derived cutoff

## Outputs

- Protein-level quantification table (pre-filtering, e.g., 4,303 proteins for PXD014546)
- Filtered protein-level quantification table (same aggregation rule after QC-score peptide filtering)
- Differential significance comparison tabulating proteins gaining vs losing significance (e.g., 45 gained / 10 lost; Mdm36 gained, Gip4 lost)

## How to apply

First run IsobaricQuant (java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o out) configured for the labeling scheme (e.g., TMT6plex, scanLevel MS3, PPMTolerance 10, topXNum 10, neutralLosses 63.98,18.01,17.03) to produce isolab.csv, isopep.csv, isopep_extra.csv, and isofrag.csv. Aggregate quantified peptides to proteins requiring at least two unique peptides per protein to yield the pre-filtering protein set (e.g., 4,303 proteins in the yeast glucose-vs-pyruvate PXD014546 dataset). Before repeating aggregation, filter peptides by the 25th percentile of a QC score — for SPS-MS3 data use peptideTopXIntensityFromTotalScore with topX = 10 to match the 10 SPS-MS3 selection notches and a recommended cutoff of 0.15 — applying the filter directly prior to protein-level quantification so protein inference is not altered. Re-aggregate the filtered peptides under the same two-unique-peptide rule, then run the differential test (e.g., glucose vs pyruvate) on both unfiltered and filtered protein tables and tabulate proteins gaining or losing significance.

## Related tools

- **IsobaricQuant** (Quantifies isobarically-labeled peptides and computes QC scores used to filter peptides before protein-level aggregation; run via CLI) — https://github.com/Villen-Lab/isobaricquant
- **Java** (Runtime (Java 17) required to execute the IsobaricQuant jar that produces the peptide quantification inputs)
- **msconvert** (Converts Thermo raw files to mzML (with vendor peak picking) that serve as IsobaricQuant input upstream of aggregation)
- **Comet** (Search engine whose PSM output is converted into the IsobaricQuant hits CSV supplying peptide-to-protein references)

## Examples

```
java -jar IsobaricQuant.jar -c tmt6plex_ms3.cfg -mzf yeast_glucose.mzML -h comet_hits.csv -o isobq_out
```

## Evaluation signals

- Pre-filtering aggregation reproduces the expected protein count (e.g., 4,303 proteins for the PXD014546 yeast glucose-vs-pyruvate dataset)
- Filtering does not change protein inference: any protein present pre-filter remains present post-filter when its two-unique-peptide requirement is still met
- Differential significance shifts match expected counts: 45 proteins gain and 10 lose significance after filtering, with Mdm36 gaining and Gip4 losing
- Protein CVs decrease after filtering out interfered peptide quantifications
- The applied cutoff equals the 25th percentile of the peptideTopXIntensityFromTotalScore distribution (starting value, ~0.15 with topX = 10 for SPS-MS3)

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- Stringent filtering substantially decreases peptide observations and can exacerbate missing values across multiple labeled sample sets
- Filtering trades accuracy/precision gains for reduced peptide observations; at the protein level the number of observed proteins is largely preserved, but low-peptide-count proteins are vulnerable
- Acquisition methods limiting PSMs per protein (e.g., 2 PSMs per protein in the PISA RTS-MS3 method) significantly limit filtering possibilities

## Evidence
