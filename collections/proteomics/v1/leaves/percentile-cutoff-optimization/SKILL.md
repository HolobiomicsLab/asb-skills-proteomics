---
name: percentile-cutoff-optimization
description: Use when filtering isobarically-labeled (iTRAQ/TMT/TMTpro) peptide quantifications
  from SPS-MS3, RTS-MS3, or MS2 runs with IsobaricQuant QC scores and a defensible,
  dataset-specific starting cutoff is needed before protein-level quantification.
when_to_use_negative:
- When the acquisition method limits PSMs per protein (e.g., the PISA RTS-MS3 method
  capped at 2 PSMs per protein), which 'significantly limited the possibility of filtering
  peptides'
- When a single universal cutoff is expected to transfer across datasets — the article
  states optimal score and cutoff 'will be dataset-dependent'
- When preserving maximum peptide observations across multiple labeled sample sets
  is the priority, since stringent filtering 'can lead to substantial loss of peptide
  observations'
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java-based tool that computes the QC scores (peptideTopXIntensityFromTotalScore,
    peptideIntensityScore, msnTotalSignal) whose distributions are used for percentile-based
    cutoff selection and filtering
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar for score generation
  repo: https://github.com/Villen-Lab/isobaricquant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:511e0c1b911a161f8594e7261dd150948873c5718e299b6e591c076ee232cf22
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - IsobaricQuant
  - Java
  - Java 17
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# percentile-cutoff-optimization

## Summary

A data-driven strategy for choosing IsobaricQuant QC-score filtering cutoffs (e.g., peptideTopXIntensityFromTotalScore with topX = 10, peptideIntensityScore, msnTotalSignal) by starting from the 25th percentile of each dataset's score distribution, then adjusting per dataset to balance improved CVs against peptide-observation loss.

## When to use

Use when filtering isobarically-labeled (iTRAQ/TMT/TMTpro) peptide quantifications from SPS-MS3, RTS-MS3, or MS2 runs with IsobaricQuant QC scores and a defensible, dataset-specific starting cutoff is needed before protein-level quantification.

## When NOT to use

- When the acquisition method limits PSMs per protein (e.g., the PISA RTS-MS3 method capped at 2 PSMs per protein), which 'significantly limited the possibility of filtering peptides'
- When a single universal cutoff is expected to transfer across datasets — the article states optimal score and cutoff 'will be dataset-dependent'
- When preserving maximum peptide observations across multiple labeled sample sets is the priority, since stringent filtering 'can lead to substantial loss of peptide observations'

## Inputs

- IsobaricQuant score tables per run (e.g., isopep_extra.csv) containing per-peptide QC scores such as peptideTopXIntensityFromTotalScore, peptideIntensityScore, and msnTotalSignal
- mzML files of iTRAQ/TMT/TMTpro-labeled MS runs (MS2 or MS3 acquisition)
- CSV file of peptide spectral matches (PSM) from a search engine such as Comet
- User-supplied IsobaricQuant text configuration file

## Outputs

- Dataset-specific filtering cutoff values for each QC score
- Filtered peptide quantification tables (interfered peptides removed prior to protein-level quantification)
- Before/after intra-protein coefficient of variation (CV) metrics and percentage of peptide observations lost (e.g., written to filtering_evaluation_results.csv)

## How to apply

Extract the per-peptide QC scores (e.g., peptideTopXIntensityFromTotalScore and msnTotalSignal) from the IsobaricQuant score tables (e.g., isopep_extra.csv) for each run and acquisition strategy. Compute the 25th percentile of the score distribution separately for each dataset as the starting cutoff — the article found this 'proved to be a good starting value for cutoff optimization'. Apply the recommended nominal values as anchors: peptideTopXIntensityFromTotalScore 0.15 with topX = 10 (matching the number of SPS-MS3 selection notches) and peptideIntensityScore 0.25 for MS3-level data, or msnTotalSignal 1e6 for MS2-level data. Filter peptides below the cutoff while applying a two-peptide minimum protection rule so proteins represented by fewer than two quantified peptides are not depleted. Compare intra-protein CVs and percentage of peptide observations lost before vs. after filtering (optionally repeating with a stringent cutoff such as msnTotalSignal < 1e7 to bound observation loss) and adjust the cutoff per dataset, since optimal score and cutoff are dataset-dependent.

## Related tools

- **IsobaricQuant** (Java-based tool that computes the QC scores (peptideTopXIntensityFromTotalScore, peptideIntensityScore, msnTotalSignal) whose distributions are used for percentile-based cutoff selection and filtering) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Runtime required to execute the IsobaricQuant jar for score generation) — https://github.com/Villen-Lab/isobaricquant

## Examples

```
java --add-opens java.base/java.lang=ALL-UNNAMED -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o out/
```

## Evaluation signals

- Intra-protein CVs decrease after filtering relative to unfiltered quantification (the benchmark task observed an approximately two-fold CV reduction with peptideTopXIntensityFromTotalScore under two-peptide minimum protection)
- The starting cutoff equals the 25th percentile of the per-run score distribution, computed separately for SPS-MS3 and RTS-MS3 (nominal peptideTopXIntensityFromTotalScore ~0.15 with topX = 10)
- Proteins represented by fewer than two quantified peptides are preserved (two-peptide minimum protection rule respected)
- Percentage of peptide observations lost is quantified for both the 25th-percentile-based cutoff and a stringent bound (e.g., msnTotalSignal < 1e7; benchmark reported losses up to 92% and 97%)
- The number of observed proteins is preserved after filtering, per the article's protein-level finding

## Limitations

- Both the optimal score and the cutoff are dataset-dependent; score distributions differed across the three datasets analyzed, so cutoffs must be re-optimized per dataset
- Filtering interfered quantifications improves accuracy and precision but 'at the cost of significantly decreased peptide observations', potentially exacerbating missing values across multiple labeled sample sets
- ms1PrecSignal is a limited predictor of reporter ion interference — a majority of scans show 100% even when quantitative accuracy is reduced — so it should not be the sole filtering score

## Evidence
