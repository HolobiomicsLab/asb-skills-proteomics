---
name: protein-inference-and-quantification-aggregation
description: Use when you have IsobaricQuant isopep/isofrag peptide reporter outputs
  from TMT/TMTpro or iTRAQ mzML data and need protein-level quantification, optionally
  filtered by QC scores (e.g. PeptideTopXIntensityFromTotalScore cutoff 0.15, topX=10)
  prior to inference.
when_to_use_negative:
- Input is already a protein-level quantification matrix — filtering operates on peptide/scan-level
  reporter intensities before inference
- Data are label-free (no iTRAQ/TMT/TMTpro reporter ions) — IsobaricQuant quantification
  and its QC scores apply to isobarically-labeled peptides
- The acquisition collected very few PSMs per protein (e.g. RTS-MS3 with max 2 PSMs
  per protein), leaving insufficient peptide redundancy for filtering to help
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: IsobaricQuant
  role: Quantifies peptide reporter ions from mzML + PSM hits, computes QC scores,
    and emits peptide-level CSVs that are filtered and aggregated to proteins
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java
  role: Runtime (Java 17 required) for the cross-platform IsobaricQuant standalone
    jar executed via command line
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Comet
  role: Search engine whose output is converted to the comma-separated hits file feeding
    IsobaricQuant quantification
- name: msconvert
  role: Converts vendor raw files to the mzML input required by IsobaricQuant (e.g.
    via the chambm/pwiz Docker image with peakPicking and zeroSamples filters)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:2060778efe3997aab7c97fddd0edbec6e842d73eee6314df8d57688bb6269a53
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
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# protein-inference-and-quantification-aggregation

## Summary

Aggregates IsobaricQuant peptide-level reporter ion intensities to protein-level quantification, deliberately applying QC-score filtering of interfered scans directly before inference so that protein identity assignment is not altered while accuracy improves.

## When to use

Use when you have IsobaricQuant isopep/isofrag peptide reporter outputs from TMT/TMTpro or iTRAQ mzML data and need protein-level quantification, optionally filtered by QC scores (e.g. PeptideTopXIntensityFromTotalScore cutoff 0.15, topX=10) prior to inference.

## When NOT to use

- Input is already a protein-level quantification matrix — filtering operates on peptide/scan-level reporter intensities before inference
- Data are label-free (no iTRAQ/TMT/TMTpro reporter ions) — IsobaricQuant quantification and its QC scores apply to isobarically-labeled peptides
- The acquisition collected very few PSMs per protein (e.g. RTS-MS3 with max 2 PSMs per protein), leaving insufficient peptide redundancy for filtering to help

## Inputs

- IsobaricQuant peptide quantification outputs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) including QC scores per peptide scan
- mzML files converted from instrument raw files via msconvert
- Comet/Percolator PSM hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- IsobaricQuant text configuration file (e.g. TMTpro isoMethod, 20 ppm PPMTolerance/MS1PPMTolerance/MS2PPMTolerance, scanLevel MS2 or RTS-MS3, topXNum=10)

## Outputs

- Protein-level aggregated reporter intensity matrix (protein x channel)
- Counts of quantified proteins (e.g. 5,380 unfiltered MS2-level; 4,713 after QC-score filtering)
- Filtered vs. unfiltered peptide tables prior to inference for comparison

## How to apply

Run IsobaricQuant (Java 17, CLI mode: java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o output) to produce isofrag.csv, isolab.csv, isopep.csv and isopep_extra.csv, then aggregate peptide reporter intensities to the protein level and count quantified proteins (PISA benchmark: 5,380 for MS2-level, 4,033 for RTS-MS3-level data). Before aggregation, optionally filter peptides using QC scores such as PeptideTopXIntensityFromTotalScore (recommended cutoff 0.15 with topX=10, matching the SPS-MS3 selection notches) or peptideIntensityScore (0.25) for MS3 data, using the 25th percentile of the score distribution as a starting value for cutoff optimization. The key decision is to place filtering immediately before protein-level quantification so protein inference is not impacted or altered in any way; on the PISA staurosporine data this reduced protein depth only from 5,380 to 4,713 while decreasing protein CVs and raising significant staurosporine targets from 4 to 7 (1 uM) and 17 to 22 (20 uM). Note that RTS-MS3 acquisitions limited to 2 PSMs per protein leave little redundancy to filter, so recommend higher PSMs per protein in RTS-MS3 methods.

## Related tools

- **IsobaricQuant** (Quantifies peptide reporter ions from mzML + PSM hits, computes QC scores, and emits peptide-level CSVs that are filtered and aggregated to proteins) — https://github.com/Villen-Lab/isobaricquant
- **Java** (Runtime (Java 17 required) for the cross-platform IsobaricQuant standalone jar executed via command line) — https://github.com/Villen-Lab/isobaricquant
- **Comet** (Search engine whose output is converted to the comma-separated hits file feeding IsobaricQuant quantification)
- **msconvert** (Converts vendor raw files to the mzML input required by IsobaricQuant (e.g. via the chambm/pwiz Docker image with peakPicking and zeroSamples filters))

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o output_folder
```

## Evaluation signals

- All four IsobaricQuant CSV outputs (isofrag.csv, isolab.csv, isopep.csv, isopep_extra.csv) exist and parse correctly
- Protein quantification counts match reported values: 5,380 proteins for MS2-level and 4,033 for RTS-MS3-level PISA data; 4,713 after QC filtering
- Filtering applied before protein-level quantification leaves the protein inference/identity list unaltered (protein count preserved, per article: filtering 'preserv[es] the number of observed proteins')
- Protein CVs decrease after removing interfered peptide quantifications relative to the unfiltered aggregation
- Downstream PISA thermal-shift analysis yields expected sensitivity gains (significant staurosporine targets 4 to 7 at 1 uM and 17 to 22 at 20 uM), with CAMKD2 accuracy and AAPK1 precision examples reproduced

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- Stringent filtering improves accuracy and precision but significantly decreases peptide observations, exacerbating missing values when multiple labeled sample sets are measured
- The PISA RTS-MS3 instrument method acquired a maximum of 2 PSMs per protein, significantly limiting the possibility of filtering peptides
- Identity of co-isolated peptides and their reporter ion release efficiency are unknown, preventing ratio decompression at inference time

## Evidence
