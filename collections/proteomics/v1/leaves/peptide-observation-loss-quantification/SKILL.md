---
name: peptide-observation-loss-quantification
description: Use when filtering MS3-level (SPS-MS3 or RTS-MS3) isobarically-labeled
  peptide quantifications with IsobaricQuant QC scores and you need to measure how
  many peptide observations are sacrificed versus the accuracy/CV gain.
when_to_use_negative:
- Input is MS2-level data, where the recommended QC score and cutoff differ (msnTotalSignal
  cutoff of 1e6 rather than peptideTopXIntensityFromTotalScore at 0.15)
- The RTS-MS3 acquisition was set to a maximum of 2 PSMs per protein, which severely
  limits filtering possibilities and makes loss quantification uninformative
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Produces the per-peptide QC scores (peptideTopXIntensityFromTotalScore, msnTotalSignal)
    in isopep_extra.csv used for filtering and loss quantification
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java
  role: Runtime required to execute the IsobaricQuant jar (Java 17) when regenerating
    score tables
  repo: https://github.com/Villen-Lab/isobaricquant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:b87f5465dbe83cfc47138c8b29d2ce8073fdc73e60fc9e0e3d8e70131ad9ec2a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - IsobaricQuant
  - Java
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# peptide-observation-loss-quantification

## Summary

Quantifies the upper bound of peptide observations lost when filtering IsobaricQuant QC-scored peptide quantifications (e.g., peptideTopXIntensityFromTotalScore ≥ 0.15 or msnTotalSignal < 1e7) under a two-peptide minimum protection rule, and verifies the resulting reduction in intra-protein CVs.

## When to use

Use when filtering MS3-level (SPS-MS3 or RTS-MS3) isobarically-labeled peptide quantifications with IsobaricQuant QC scores and you need to measure how many peptide observations are sacrificed versus the accuracy/CV gain.

## When NOT to use

- Input is MS2-level data, where the recommended QC score and cutoff differ (msnTotalSignal cutoff of 1e6 rather than peptideTopXIntensityFromTotalScore at 0.15)
- The RTS-MS3 acquisition was set to a maximum of 2 PSMs per protein, which severely limits filtering possibilities and makes loss quantification uninformative

## Inputs

- isopep_extra.csv score tables from IsobaricQuant runs (SPS-MS3 and RTS-MS3), containing peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide
- Peptide-to-protein group mapping for applying the two-peptide minimum protection rule

## Outputs

- filtering_evaluation_results.csv with per-strategy intra-protein CVs before/after filtering, percentage of peptide observations lost, and cutoff values
- Comparison of peptide observation losses under the nominal (0.15) and stringent (msnTotalSignal < 1e7) filters

## How to apply

Load the isopep_extra.csv score tables for the SPS-MS3 and RTS-MS3 runs, extracting peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide. Compute the 25th percentile of the peptideTopXIntensityFromTotalScore distribution separately for each acquisition strategy as the starting cutoff (the paper recommends a nominal value of 0.15 with topX = 10, matching the number of SPS-MS3 selection notches). Filter peptides below the cutoff while applying the two-peptide minimum protection rule so proteins represented by fewer than two quantified peptides are not depleted. Repeat with the stringent msnTotalSignal < 1e7 cutoff to bound the worst-case observation loss, then compute intra-protein coefficients of variation before and after filtering and the percentage of peptide observations lost. Correct application is judged by recovering the reported ~two-fold reduction in intra-protein CVs and the stringent-filter observation losses (up to 92% and 97%).

## Related tools

- **IsobaricQuant** (Produces the per-peptide QC scores (peptideTopXIntensityFromTotalScore, msnTotalSignal) in isopep_extra.csv used for filtering and loss quantification) — https://github.com/Villen-Lab/isobaricquant
- **Java** (Runtime required to execute the IsobaricQuant jar (Java 17) when regenerating score tables) — https://github.com/Villen-Lab/isobaricquant

## Examples

```
java -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o output_folder  # then filter peptides in isopep_extra.csv where peptideTopXIntensityFromTotalScore < 0.15, protecting proteins with < 2 peptides
```

## Evaluation signals

- Intra-protein CVs decrease approximately two-fold after peptideTopXIntensityFromTotalScore filtering with two-peptide minimum protection
- Peptide observation losses under the stringent msnTotalSignal < 1e7 filter reach up to 92% and 97% for SPS-MS3 and RTS-MS3 respectively
- Starting cutoff equals the 25th percentile of the per-strategy peptideTopXIntensityFromTotalScore distribution (~0.15 nominal)
- Proteins with fewer than two quantified peptides retain their peptides (two-peptide protection invariant holds)
- All computed metrics are written to filtering_evaluation_results.csv with one row per acquisition strategy

## Limitations

- Filtering improves accuracy and precision at the cost of substantially decreased peptide observations, exacerbating missing values when multiple labeled sample sets are measured
- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting point for optimization
- Stringent filtering can deplete peptide observations badly enough that a two-peptide minimum protection rule is needed to avoid losing proteins entirely

## Evidence
