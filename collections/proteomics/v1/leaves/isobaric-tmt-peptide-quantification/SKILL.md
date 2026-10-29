---
name: isobaric-tmt-peptide-quantification
description: Use when quantifying TMT, TMTpro, or iTRAQ reporter ions from mzML files
  with a search-engine PSM table, especially when co-isolation interference may distort
  ratios and scan-level filtering is desired.
when_to_use_negative:
- The data are not isobarically labeled (e.g. label-free or SILAC quantification)
  or labels are outside iTRAQ 8-plex / TMT 11-plex / TMTpro 16-plex
- Only a protein-level feature table exists and scan-level reporter-ion quantification
  cannot be re-run from mzML and PSM hits
- RTS-MS3 acquisition collected very few PSMs per protein (e.g. max 2), leaving too
  little redundancy for filtering to help
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: IsobaricQuant
  role: 'Core tool: quantifies reporter ions from mzML + hits, computes QC scores,
    and supports filtering of interfered scans'
  repo: https://github.com/Villen-Lab/isobaricquant
- name: msconvert
  role: Converts vendor raw files to mzML input for IsobaricQuant
  repo: http://proteowizard.sourceforge.net/tools/filters.html
- name: Comet
  role: Search engine whose PSM output is converted into the IsobaricQuant hits CSV
  repo: https://uwpr.github.io/Comet/
- name: Java 17
  role: Runtime required to execute the IsobaricQuant jar in GUI or CLI mode
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202100253
    title: IsobaricQuant enables cross‐platform quantification, visualization, and
      filtering of isobarically‐labeled peptides
  source_content_hash: sha256:f7bc45db7849cfe874ae8692540ee151c754ef2282221280354328f121cdec41
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
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# isobaric-tmt-peptide-quantification

## Summary

Quantify, visualize, and filter isobarically-labeled (iTRAQ/TMT/TMTpro) peptides with IsobaricQuant, a Java 17 standalone tool that computes per-scan QC scores to remove reporter-ion interference at MS2 and MS3 levels prior to protein-level aggregation.

## When to use

Use when quantifying TMT, TMTpro, or iTRAQ reporter ions from mzML files with a search-engine PSM table, especially when co-isolation interference may distort ratios and scan-level filtering is desired.

## When NOT to use

- The data are not isobarically labeled (e.g. label-free or SILAC quantification) or labels are outside iTRAQ 8-plex / TMT 11-plex / TMTpro 16-plex
- Only a protein-level feature table exists and scan-level reporter-ion quantification cannot be re-run from mzML and PSM hits
- RTS-MS3 acquisition collected very few PSMs per protein (e.g. max 2), leaving too little redundancy for filtering to help

## Inputs

- Thermo raw file converted to mzML (e.g. via msconvert with peakPicking true 1- and zeroSamples removeExtra)
- CSV hits file of PSMs from a search engine such as Comet (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z)
- User-supplied IsobaricQuant text config file (label set, scan level, PPMTolerance, topXNum, neutral losses)
- Optional reporter ion isotopic impurity correction CSV

## Outputs

- isolab.csv (isolation-window QC)
- isopep.csv (per-peptide reporter intensities)
- isopep_extra.csv (per-PSN QC scores: ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore)
- isofrag.csv (fragment ion assignment)
- Filtered protein-level quantification after QC-score-based peptide removal

## How to apply

Convert Thermo raw files to mzML (e.g. with msconvert, peak picking true 1-, zeroSamples removeExtra) and search them with an engine such as Comet, converting the search output into a comma-separated hits file (Search ID, Peptide ID, Sequence, Reference, Charge, Start Scan, M/Z). Write a config specifying the label set (iTRAQ up to 8-plex, TMT up to 11-plex, TMTpro up to 16-plex), scan level (MS2 or MS3), and tolerance (e.g. PPMTolerance 10 or 20), then run IsobaricQuant via `java -jar IsobaricQuant.jar -c config -mzf mzML -h hits -o out`, which produces isolab.csv, isopep.csv, isopep_extra.csv, and isofrag.csv. Inspect per-PSN QC scores (ms1PrecSignal, msnTotalSignal, peptideIntensityScore, peptideTopXIntensityFromTotalScore) in isopep_extra.csv and filter interfered scans before protein aggregation — this placement assures protein inference is not altered. Use recommended cutoffs as starting points: msnTotalSignal 1e6 for MS2; peptideIntensityScore 0.25 and peptideTopXIntensityFromTotalScore 0.15 (topX = 10, matching SPS notches) for MS3, starting from the 25th percentile and tuning per dataset. Expect improved accuracy and lower protein CVs at the cost of fewer peptide observations, while the number of observed proteins is largely preserved (e.g. 4,303 yeast proteins before filtering).

## Related tools

- **IsobaricQuant** (Core tool: quantifies reporter ions from mzML + hits, computes QC scores, and supports filtering of interfered scans) — https://github.com/Villen-Lab/isobaricquant
- **msconvert** (Converts vendor raw files to mzML input for IsobaricQuant) — http://proteowizard.sourceforge.net/tools/filters.html
- **Comet** (Search engine whose PSM output is converted into the IsobaricQuant hits CSV) — https://uwpr.github.io/Comet/
- **Java 17** (Runtime required to execute the IsobaricQuant jar in GUI or CLI mode)

## Examples

```
java -jar IsobaricQuant.jar -c config -mzf run.mzML -h hits.csv -o out
```

## Evaluation signals

- Protein counts before vs. after filtering remain comparable (e.g. 4,303 proteins in the yeast glucose-vs-pyruvate dataset pre-filtering)
- Protein CVs decrease after removing interfered peptide quantifications
- Expected output files exist and parse: isolab.csv, isopep.csv, isopep_extra.csv, isofrag.csv
- Filtered vs. unfiltered differential testing shows plausible significance shifts (yeast TMT6plex benchmark: 45 proteins gained, 10 lost significance, Mdm36 gained and Gip4 lost)
- QC score distributions match known patterns, e.g. ms1PrecSignal saturation at 100% for up to 60% of precursors despite reduced accuracy

## Limitations

- Optimal QC score and cutoff are dataset-dependent; 25th percentile is only a starting point for tuning
- Filtering improves accuracy and precision but significantly decreases peptide observations and can exacerbate missing values across multiple labeled sets
- ms1PrecSignal has limited predictive value — a majority of scans show 100% even when quantification is impacted
- Identity of co-isolated peptides and their reporter ion release efficiency are unknown, so ratio decompression cannot be performed
- RTS-MS3 methods limited to few PSMs per protein leave little room for filtering

## Evidence
