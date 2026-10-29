---
name: intra-protein-cv-precision-analysis
description: 'Use when comparing SPS-MS3 or RTS-MS3 isobaric peptide quantification
  precision: you have IsobaricQuant score tables (isopep_extra.csv) and want to know
  whether QC-score filtering reduces intra-protein CVs without depleting protein coverage.'
when_to_use_negative:
- The PISA RTS-MS3 instrument method had been set to acquire a maximum of 2 PSMs per
  protein, leaving no filtering headroom — CV analysis on such maximally 2-PSM-per-protein
  data cannot show meaningful filtering effects
- Input is raw mzML or search-engine PSM output (e.g., Comet hits) rather than post-IsobaricQuant
  score tables; run IsobaricQuant quantification first
- MS2-level data, where the article recommends msnTotalSignal with a 1e6 cutoff instead
  of MS3 peptide-intensity scores
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Generates the QC scores (peptideTopXIntensityFromTotalScore, msnTotalSignal)
    from iTRAQ/TMT mzML and PSM input files that are filtered and evaluated for intra-protein
    CVs
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java
  role: Runtime required to execute the IsobaricQuant jar (Java 17) via GUI or command
    line
  repo: https://github.com/Villen-Lab/isobaricquant
- name: IsobaricQuant benchmark
  role: Repository containing the scripts that generate the CV/filtering benchmark
    figures and analyses
  repo: https://github.com/Villen-Lab/isobaricquant_benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:f769531150ac3e9c977e242f60f78eb89aeb3044171cea4a2b38c48090f5d95f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Villen-Lab/isobaricquant
  tools:
  - IsobaricQuant
  - IsobaricQuant benchmark
  - Java
  edam_operation: http://edamontology.org/operation_2945
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

# intra-protein-cv-precision-analysis

## Summary

Computes intra-protein coefficients of variation (CVs) for SPS-MS3 and RTS-MS3 peptide quantifications before and after QC-score filtering (e.g., peptideTopXIntensityFromTotalScore cutoff ~0.15, topX = 10) to quantify the precision gain from removing interfered quantifications, while protecting proteins with fewer than two quantified peptides.

## When to use

Use when comparing SPS-MS3 or RTS-MS3 isobaric peptide quantification precision: you have IsobaricQuant score tables (isopep_extra.csv) and want to know whether QC-score filtering reduces intra-protein CVs without depleting protein coverage.

## When NOT to use

- The PISA RTS-MS3 instrument method had been set to acquire a maximum of 2 PSMs per protein, leaving no filtering headroom — CV analysis on such maximally 2-PSM-per-protein data cannot show meaningful filtering effects
- Input is raw mzML or search-engine PSM output (e.g., Comet hits) rather than post-IsobaricQuant score tables; run IsobaricQuant quantification first
- MS2-level data, where the article recommends msnTotalSignal with a 1e6 cutoff instead of MS3 peptide-intensity scores

## Inputs

- isopep_extra.csv IsobaricQuant score tables for SPS-MS3 and RTS-MS3 runs (per-peptide peptideTopXIntensityFromTotalScore and msnTotalSignal values)
- Peptide-to-protein-group assignments for grouping peptides into protein CVs

## Outputs

- filtering_evaluation_results.csv with intra-protein CVs before and after filtering per acquisition strategy
- Percentage of peptide observations lost per cutoff (25th-percentile-based and stringent msnTotalSignal < 1e7)
- Cutoff values used per acquisition strategy (e.g., peptideTopXIntensityFromTotalScore 0.15, topX = 10)

## How to apply

Load the isopep_extra.csv score tables for the SPS-MS3 and RTS-MS3 runs and extract peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide. Compute the 25th percentile of the peptideTopXIntensityFromTotalScore distribution separately for each acquisition strategy as the starting cutoff (the recommended nominal value is 0.15 with topX = 10, set to equal the number of SPS-MS3 selection notches). Filter peptides below the cutoff while applying the two-peptide minimum protection rule so proteins represented by fewer than two quantified peptides are not depleted. Repeat with a stringent msnTotalSignal < 1e7 cutoff to quantify the upper bound of peptide observation loss, then compute intra-protein CVs before and after filtering for each strategy and the percentage of peptide observations lost, writing all metrics to filtering_evaluation_results.csv.

## Related tools

- **IsobaricQuant** (Generates the QC scores (peptideTopXIntensityFromTotalScore, msnTotalSignal) from iTRAQ/TMT mzML and PSM input files that are filtered and evaluated for intra-protein CVs) — https://github.com/Villen-Lab/isobaricquant
- **Java** (Runtime required to execute the IsobaricQuant jar (Java 17) via GUI or command line) — https://github.com/Villen-Lab/isobaricquant
- **IsobaricQuant benchmark** (Repository containing the scripts that generate the CV/filtering benchmark figures and analyses) — https://github.com/Villen-Lab/isobaricquant_benchmark

## Examples

```
java --add-opens java.base/java.lang=ALL-UNNAMED -jar IsobaricQuant.jar -c <config_file> -mzf <mz_file> -h <hits_file> -o <output_folder>
```

## Evaluation signals

- Expected outcome: filtering SPS-MS3 and RTS-MS3 peptide quantifications with peptideTopXIntensityFromTotalScore (two-peptide minimum protection) decreases intra-protein CVs by approximately two-fold
- Under the stringent msnTotalSignal < 1e7 cutoff, peptide observation losses should approach the upper bounds of ~92% (SPS-MS3) and ~97% (RTS-MS3)
- The chosen peptideTopXIntensityFromTotalScore cutoff should be near the 25th percentile of the per-strategy score distribution (nominal 0.15 with topX = 10)
- At the protein level, the number of observed proteins should be preserved after filtering despite reduced peptide observations
- Proteins represented by fewer than two quantified peptides must remain unfiltered (two-peptide minimum protection invariant in filtering_evaluation_results.csv)

## Limitations

- Both the optimal QC score and the cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization, not a universal threshold
- Filtering interfered quantifications improves accuracy and precision but significantly decreases peptide observations, which can exacerbate missing values when multiple sets of labeled samples are measured
- Stringent filtering can deplete up to 92%/97% of peptide observations under the msnTotalSignal < 1e7 cutoff; missing values can be mitigated by increasing peptide coverage, e.g., via extensive fractionation

## Evidence

- [abstract] its QC scores can successfully filter out scans with reduced quantitative accuracy at MS2 and MS3 levels, removing inaccurate peptide quantifications and decreasing protein CVs: "its QC scores can successfully filter out scans with reduced quantitative accuracy at MS2 and MS3 levels, removing inaccurate peptide quantifications and decreasing protein CVs"
