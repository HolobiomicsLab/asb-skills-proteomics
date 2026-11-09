---
name: isobaric-peptide-qc-score-filtering
description: Use when TMT/iTRAQ PSM quantification tables are impaired by precursor
  co-isolation (ratio compression) and you need to remove inaccurate peptide quantifications
  at MS2 or MS3 level before protein-level aggregation.
when_to_use_negative:
- When the RTS-MS3 acquisition method was set to a maximum of 2 PSMs per protein,
  since this significantly limits the possibility of filtering peptides
- When filtering would be applied after protein-level quantification — the article
  states filtering must occur directly prior to protein-level quantification so protein
  inference is not impacted
- When ms1PrecSignal is the only available QC score, since a majority of scans show
  100% even when quantitative accuracy is reduced
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: Java-based standalone tool that computes QC scores (peptideTopXIntensityFromTotalScore,
    msnTotalSignal, ms1PrecSignal) and filters interfered isobarically-labeled peptide
    quantifications at MS2 and MS3 levels
  repo: https://github.com/Villen-Lab/isobaricquant
- name: Java 17
  role: Required runtime to execute the IsobaricQuant jar
- name: msconvert
  role: Converts instrument raw files to mzML with recommended filters (peakPicking
    true 1-, zeroSamples removeExtra) before IsobaricQuant quantification
- name: Comet
  role: Search engine whose output is converted into the PSM hits CSV input required
    by IsobaricQuant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:df0a9891ffc6e3760bd559c055f72bc1c6e661a1d7c45cdbc01761c22952a42b
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
  edam_operation: http://edamontology.org/operation_3634
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

# isobaric-peptide-qc-score-filtering

## Summary

Filter isobarically-labeled (TMT/iTRAQ/TMTpro) peptide quantifications suffering from co-isolation interference using IsobaricQuant QC scores (e.g., peptideTopXIntensityFromTotalScore with cutoff 0.15 at MS3 level), while applying a two-peptide minimum protection rule so protein-level CVs decrease without depleting protein observations.

## When to use

Use when TMT/iTRAQ PSM quantification tables are impaired by precursor co-isolation (ratio compression) and you need to remove inaccurate peptide quantifications at MS2 or MS3 level before protein-level aggregation.

## When NOT to use

- When the RTS-MS3 acquisition method was set to a maximum of 2 PSMs per protein, since this significantly limits the possibility of filtering peptides
- When filtering would be applied after protein-level quantification — the article states filtering must occur directly prior to protein-level quantification so protein inference is not impacted
- When ms1PrecSignal is the only available QC score, since a majority of scans show 100% even when quantitative accuracy is reduced

## Inputs

- mzML file(s) converted from instrument raw files (via msconvert with peakPicking and zeroSamples filters)
- CSV file of peptide spectral matches (PSMs) from a search engine such as Comet
- User-supplied IsobaricQuant text configuration file
- Optional reporter ion isotopic impurity correction CSV
- IsobaricQuant output score tables (e.g., isopep_extra.csv) containing peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide

## Outputs

- Filtered peptide quantification table with interfered scans removed prior to protein-level quantification
- Per-peptide QC scores (peptideTopXIntensityFromTotalScore, peptideIntensityScore, msnTotalSignal, ms1PrecSignal)
- Intra-protein CVs before and after filtering (e.g., written to filtering_evaluation_results.csv)
- Percentage of peptide observations lost under each cutoff

## How to apply

Run IsobaricQuant (Java 17, jar via command line: java -jar IsobaricQuant.jar -c config -mzf mzML -h hits_file -o output) on mzML files converted with msconvert plus a search-engine PSM CSV (e.g., Comet output) to obtain QC scores such as peptideTopXIntensityFromTotalScore and msnTotalSignal per peptide. For MS3-level (SPS-MS3, RTS-MS3) data, compute the 25th percentile of the peptideTopXIntensityFromTotalScore distribution as a starting cutoff (recommended nominal value 0.15 with topX = 10, matching the number of SPS-MS3 selection notches); for MS2-level data, the recommended msnTotalSignal cutoff is 1e6. Filter peptides below the cutoff directly prior to protein-level quantification so protein inference is not altered, and protect proteins represented by fewer than two quantified peptides from depletion. Because cutoffs are dataset-dependent, verify improvement by comparing intra-protein coefficients of variation before and after filtering, and quantify the cost as percentage of peptide observations lost (stringent msnTotalSignal < 1e7 filtering can lose up to 92% and 97% of observations for SPS-MS3 and RTS-MS3, while still achieving roughly two-fold CV reduction).

## Related tools

- **IsobaricQuant** (Java-based standalone tool that computes QC scores (peptideTopXIntensityFromTotalScore, msnTotalSignal, ms1PrecSignal) and filters interfered isobarically-labeled peptide quantifications at MS2 and MS3 levels) — https://github.com/Villen-Lab/isobaricquant
- **Java 17** (Required runtime to execute the IsobaricQuant jar)
- **msconvert** (Converts instrument raw files to mzML with recommended filters (peakPicking true 1-, zeroSamples removeExtra) before IsobaricQuant quantification)
- **Comet** (Search engine whose output is converted into the PSM hits CSV input required by IsobaricQuant)

## Examples

```
java --add-opens java.base/java.lang=ALL-UNNAMED -jar IsobaricQuant.jar -c isobaricquant_config.txt -mzf run.mzML -h comet_hits.csv -o out
```

## Evaluation signals

- Intra-protein CVs decrease approximately two-fold after filtering SPS-MS3 and RTS-MS3 quantifications with peptideTopXIntensityFromTotalScore under the two-peptide minimum protection rule
- The number of observed proteins is preserved at the protein level despite peptide-level filtering
- Percentage of peptide observations lost is reported; expect up to 92% (SPS-MS3) and 97% (RTS-MS3) loss under the stringent msnTotalSignal < 1e7 cutoff
- Starting cutoff equals the 25th percentile of the score distribution; recommended nominal MS3 cutoff for peptideTopXIntensityFromTotalScore is 0.15 with topX = 10
- IsobaricQuant quantifications agree with commonly used software when compared head-to-head

## Limitations

- Optimal QC score and cutoff are dataset-dependent; the 25th percentile is only a starting value for cutoff optimization
- Stringent filtering leads to substantial loss of peptide observations, exacerbating missing values when multiple sets of labeled samples are measured
- Identity of co-isolated peptides and their reporter ion release efficiency remain unknown, so ratio decompression is not possible
- Missing values can be mitigated by increasing peptide coverage, e.g., via extensive fractionation, or by setting RTS-MS3 acquisition to higher PSMs per protein

## Evidence
