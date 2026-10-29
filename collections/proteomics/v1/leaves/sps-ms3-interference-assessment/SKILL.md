---
name: sps-ms3-interference-assessment
description: Use when TMT/TMTpro iTRAQ data acquired with SPS-MS3 or RTS-MS3 shows
  ratio compression from co-isolated co-eluting precursors, and you need to filter
  interfered peptide quantifications (isopep_extra.csv score tables) prior to protein-level
  quantification.
when_to_use_negative:
- When the acquisition method limits peptides to a maximum of 2 PSMs per protein (e.g.,
  the PISA RTS-MS3 method), filtering has little room to act without depleting proteins
- When the goal is MS2-level quantification filtering with msnTotalSignal — the recommended
  MS2 cutoff is 1e6, and score behavior differs from the MS3 peptide-level scores
  used here
- 'When cutoffs from this dataset are applied blindly to a new dataset: the optimal
  score and cutoff are dataset-dependent and must be re-derived (e.g., from the 25th
  percentile)'
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java-based tool that computes QC scores (peptideTopXIntensityFromTotalScore,
    msnTotalSignal) from mzML + PSM hit files; its scores are used to filter interfered
    SPS-MS3/RTS-MS3 peptide quantifications
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar
- name: msconvert
  role: Converts instrument raw files to mzML, the spectral input format required
    by IsobaricQuant
- name: Comet
  role: Search engine whose output file is converted to the PSM hits file consumed
    by IsobaricQuant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:fc7cd21e0e73ddebfdd7eb9a747122018d80d674a00ea4f24db72f9c42f81c29
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
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sps-ms3-interference-assessment

## Summary

Assesses precursor interference in SPS-MS3 (and RTS-MS3) isobarically-labeled peptide quantifications using IsobaricQuant QC scores — especially peptideTopXIntensityFromTotalScore (topX=10, recommended cutoff 0.15, seeded from the 25th percentile) — to filter inaccurate peptide quantifications and reduce intra-protein CVs while protecting proteins with fewer than two peptides.

## When to use

Use when TMT/TMTpro iTRAQ data acquired with SPS-MS3 or RTS-MS3 shows ratio compression from co-isolated co-eluting precursors, and you need to filter interfered peptide quantifications (isopep_extra.csv score tables) prior to protein-level quantification.

## When NOT to use

- When the acquisition method limits peptides to a maximum of 2 PSMs per protein (e.g., the PISA RTS-MS3 method), filtering has little room to act without depleting proteins
- When the goal is MS2-level quantification filtering with msnTotalSignal — the recommended MS2 cutoff is 1e6, and score behavior differs from the MS3 peptide-level scores used here
- When cutoffs from this dataset are applied blindly to a new dataset: the optimal score and cutoff are dataset-dependent and must be re-derived (e.g., from the 25th percentile)

## Inputs

- isopep_extra.csv IsobaricQuant score tables for SPS-MS3 and RTS-MS3 runs (peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide)
- mzML files converted from instrument raw data (e.g., via msconvert with peakPicking and zeroSamples filters)
- CSV of peptide spectral matches (PSMs) from a search engine such as Comet
- IsobaricQuant text configuration file (topX=10 for peptideTopXIntensityFromTotalScore)

## Outputs

- Filtered per-peptide quantification tables retaining peptides above the QC cutoff with two-peptide minimum protection
- Intra-protein coefficient of variation (CV) values before and after filtering, per acquisition strategy
- Percentage of peptide observations lost per filter condition
- filtering_evaluation_results.csv containing the computed metrics

## How to apply

Load the isopep_extra.csv score tables produced by IsobaricQuant for the SPS-MS3 and RTS-MS3 runs, extracting peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide. Compute the 25th percentile of the peptideTopXIntensityFromTotalScore distribution separately for each acquisition strategy as the starting cutoff, then refine toward the recommended nominal value of 0.15 with topX = 10 (matching the number of SPS-MS3 selection notches). Filter peptides below the cutoff while applying the two-peptide minimum protection rule so proteins represented by fewer than two quantified peptides are not depleted, then optionally repeat with the stringent msnTotalSignal < 1e7 cutoff to bound the maximum peptide-observation loss. Compute intra-protein coefficients of variation before and after filtering per acquisition strategy and report the percentage of peptide observations lost; expect roughly a two-fold CV reduction, with observation losses up to 92% (SPS-MS3) and 97% (RTS-MS3) under the stringent filter.

## Related tools

- **IsobaricQuant** (Java-based tool that computes QC scores (peptideTopXIntensityFromTotalScore, msnTotalSignal) from mzML + PSM hit files; its scores are used to filter interfered SPS-MS3/RTS-MS3 peptide quantifications) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Runtime required to execute the IsobaricQuant jar)
- **msconvert** (Converts instrument raw files to mzML, the spectral input format required by IsobaricQuant)
- **Comet** (Search engine whose output file is converted to the PSM hits file consumed by IsobaricQuant)

## Examples

```
java --add-opens java.base/java.lang=ALL-UNNAMED -jar IsobaricQuant.jar -c config.txt -mzf run.mzML -h hits.csv -o output_folder
```

## Evaluation signals

- Intra-protein CVs decrease approximately two-fold after applying the peptideTopXIntensityFromTotalScore filter to SPS-MS3 and RTS-MS3 quantifications
- The starting cutoff equals the 25th percentile of the peptideTopXIntensityFromTotalScore distribution, computed separately per acquisition strategy, converging near the recommended nominal 0.15 with topX = 10
- Proteins represented by fewer than two quantified peptides are retained (two-peptide minimum protection), so protein-group membership is not depleted by filtering
- Stringent msnTotalSignal < 1e7 filtering yields upper-bound observation losses of up to 92% (SPS-MS3) and 97% (RTS-MS3), quantified as the percentage of peptide observations lost
- All metrics (CVs before/after, percentage lost) are written to filtering_evaluation_results.csv for verification

## Limitations

- Stringent filtering leads to substantial loss of peptide observations (up to 92–97% under msnTotalSignal < 1e7), potentially exacerbating missing values when multiple labeled sample sets are measured
- Both the optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for optimization
- ms1PrecSignal is a limited predictor of reporter ion interference — a majority of scans show 100% even when quantitative accuracy is reduced
- Filtering removes inaccurate quantifications at the cost of decreased peptide observations, although protein-level observation counts are largely preserved

## Evidence
