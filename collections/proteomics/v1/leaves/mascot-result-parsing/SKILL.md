---
name: mascot-result-parsing
description: Use when Mascot database-search results (in .xml for PTM-localization
  validation or .csv for peptide quantification) must be parsed, filtered, and converted
  into Brno nomenclature combinatorial histone marks.
when_to_use_negative:
- Input is raw vendor or mzML spectra rather than a Mascot database-search result
  — the tools require Mascot .xml or .csv outputs
- PTM assignments lack site-determining ions on both sides of the modified position
  and cannot be unambiguously localized
- The goal is bottom-up peptide quantification across many different sequences rather
  than middle-down/top-down comparison of histone tail sequences with scrambled combinatorial
  PTMs
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Histone Coder
  role: Java tool that parses Mascot .xml output, counts site-determining MS/MS ions
    to validate PTM localization, and converts results to Brno nomenclature
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: isoScale
  role: Java tool that parses the Histone Coder result file plus Mascot .csv output
    to quantify peptides from total MS/MS fragment-ion intensity, including FIRR-based
    quantification of co-fragmenting isobaric peptides
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- name: Mascot
  role: Database-searching engine whose assigned PTM localizations and output files
    (.xml, .csv) are the parsed input
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:69d932553b6b7e42176c7edbd751e4244c869b750b8f0cc5aa455ccac75846eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mascot-result-parsing

## Summary

Parse Mascot (Matrix Science) database-search outputs with the middle-down Proteomics tools: Histone Coder reads Mascot .xml to validate PTM localization via site-determining c/z ions and converts identifications to Brno nomenclature, while isoScale reads the Histone Coder result file plus Mascot .csv to quantify peptides from MS/MS total fragment-ion intensities.

## When to use

Use when Mascot database-search results (in .xml for PTM-localization validation or .csv for peptide quantification) must be parsed, filtered, and converted into Brno nomenclature combinatorial histone marks.

## When NOT to use

- Input is raw vendor or mzML spectra rather than a Mascot database-search result — the tools require Mascot .xml or .csv outputs
- PTM assignments lack site-determining ions on both sides of the modified position and cannot be unambiguously localized
- The goal is bottom-up peptide quantification across many different sequences rather than middle-down/top-down comparison of histone tail sequences with scrambled combinatorial PTMs

## Inputs

- Mascot database-search output in .xml format (input to Histone Coder)
- Mascot .csv output data file (input to isoScale)
- Histone Coder result file with validated PTM localizations (input to isoScale)
- User-selected filter settings: spectrum score, fragment ion types, MS/MS tolerance, fragmentation mode (CID/HCD or ETD)

## Outputs

- Histone Coder result file listing site-determining ion counts and types per PTM in Brno nomenclature (e.g. K4me3K14ac)
- isoScale quantification table of peptide abundances (total MS/MS fragment-ion intensity, FIRR-apportioned intensities for isobaric pairs)

## How to apply

Feed Mascot .xml output into Histone Coder, which counts site-determining c and z fragment ions between the Mascot-assigned PTM localization and the closest alternative host residues, accepting only PTMs with site-determining ions on each side to eliminate false-positive localizations; PTMs handled include phosphorylation (S,T,Y), acetylation (K), mono-/dimethylation (K,R) and trimethylation (K). Apply the interface filters for spectrum score, fragment ion types and MS/MS tolerance, then write the Histone Coder result file with identifications converted to Brno nomenclature (e.g. K4me3K14ac). Next, load that result file together with the Mascot .csv data file into isoScale, select the fragmentation mode (CID/HCD or ETD) and the MS/MS ion tolerance, and compute total fragment-ion intensity per spectrum, using FIRR for composite spectra of co-fragmenting isobaric peptides. The rationale is that Mascot alone does not guarantee unambiguous PTM localization, so ion-based validation must precede intensity-based quantification.

## Related tools

- **Histone Coder** (Java tool that parses Mascot .xml output, counts site-determining MS/MS ions to validate PTM localization, and converts results to Brno nomenclature) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **isoScale** (Java tool that parses the Histone Coder result file plus Mascot .csv output to quantify peptides from total MS/MS fragment-ion intensity, including FIRR-based quantification of co-fragmenting isobaric peptides) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
- **Mascot** (Database-searching engine whose assigned PTM localizations and output files (.xml, .csv) are the parsed input)

## Examples

```
java -jar "Histone Coder.jar"  # then run: java -jar isoScale-v2.jar with the Histone Coder result file and Mascot .csv data file
```

## Evaluation signals

- Reproduce the Fig 2B reference output: the H3K27me2 example reports 2 site-determining ions on the left side and 8 on the right side
- Every accepted PTM has at least one site-determining ion on each side of the assigned position (unambiguous localization criterion)
- Output identifications are expressed in Brno nomenclature (e.g. K4me3K14ac)
- For FIRR-quantified isobaric pairs, the two non-negative relative abundances together account for the composite spectrum's total ion intensity
- Per-histone intensities normalize to percentage abundances summing to 100% of all quantified peptides

## Limitations

- Histone Coder requires Java and accepts only Mascot .xml output; isoScale requires Mascot .csv output, so raw search results in other formats must be converted first
- FIRR-based apportionment of composite spectra relies on localization-diagnostic fragment ions being detected within the chosen MS/MS ion tolerance; no numeric acceptance threshold is reported in the paper
- The tools are recommended for middle-down or top-down histone analyses and the included PTM set is limited to phosphorylation (S,T,Y), acetylation (K), mono-/dimethylation (K,R) and trimethylation (K)

## Evidence

- [readme] Input formats: [evidence span withheld — anchor status: external_doc]
