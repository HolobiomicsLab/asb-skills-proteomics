---
name: cz-fragment-ion-series-interpretation
description: Use when Mascot database-search results (.xml) from ETD middle-down histone
  MS/MS must have PTM localizations confirmed by site-determining c/z ions before
  downstream quantification of combinatorial marks.
when_to_use_negative:
- Input is Mascot .csv output rather than .xml — Histone Coder requires the .xml Mascot
  output; .csv files are handled downstream by isoScale
- Bottom-up peptide-level data where standard Mascot site localization already suffices;
  this skill targets hypermodified histone tails with isobaric/cofragmenting PTM positional
  variants in middle-down or top-down workflows
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Histone Coder
  role: Java tool that counts site-determining MS/MS ions to validate Mascot PTM localizations
    and converts output to Brno nomenclature
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database-search engine whose .xml output provides the initial PTM localizations
    to be validated
- name: isoScale
  role: Downstream quantification tool consuming the Histone Coder result file to
    compute FIRR for isobaric peptides
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:e6cf3de863e974d8a76ba814ceb2d29b9a3cf083b820c0236c1cc88f5306240d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  - isoScale
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cz-fragment-ion-series-interpretation

## Summary

Validates Mascot-assigned PTM localizations in middle-down histone MS/MS data by counting site-determining c and z fragment ions on both sides of each modification, eliminating false-positive assignments before peptide quantification with isoScale.

## When to use

Use when Mascot database-search results (.xml) from ETD middle-down histone MS/MS must have PTM localizations confirmed by site-determining c/z ions before downstream quantification of combinatorial marks.

## When NOT to use

- Input is Mascot .csv output rather than .xml — Histone Coder requires the .xml Mascot output; .csv files are handled downstream by isoScale
- Bottom-up peptide-level data where standard Mascot site localization already suffices; this skill targets hypermodified histone tails with isobaric/cofragmenting PTM positional variants in middle-down or top-down workflows

## Inputs

- Mascot database-search output file in .xml extension containing peptide-spectrum matches with PTM assignments
- Raw MS/MS spectra from ETD (electron transfer dissociation) middle-down acquisitions of hypermodified histone tails
- User-selected filter settings: spectrum score, fragment ion types, MS/MS tolerance

## Outputs

- Histone Coder result file listing each combinatorial mark (Brno nomenclature) with the number and type of site-determining ions between the assigned PTM localization and the closest alternative host residues
- Validated PTM-localized peptide identifications suitable as input to isoScale for FIRR-based quantification

## How to apply

Load the Mascot output (.xml) into the Java-based Histone Coder and, for each Mascot-assigned PTM, locate the nearest other amino acids in the histone sequence that can host the same modification. Count site-determining c and z fragment ions in the MS/MS spectrum on both the left and right sides of each alternative site, using the interface filters for spectrum score, fragment ion types (ETD c/z ions) and MS/MS tolerance. A PTM assignment is accepted only when site-determining ions are present on both sides of the assigned position, ensuring unambiguous localization. Finally convert the validated Mascot identifications to standard Brno nomenclature (e.g. K4me3K14ac) and write the result file, which also serves as the input result file for isoScale quantification.

## Related tools

- **Histone Coder** (Java tool that counts site-determining MS/MS ions to validate Mascot PTM localizations and converts output to Brno nomenclature) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database-search engine whose .xml output provides the initial PTM localizations to be validated)
- **isoScale** (Downstream quantification tool consuming the Histone Coder result file to compute FIRR for isobaric peptides) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true

## Examples

```
java -jar "Histone Coder.jar"
```

## Evaluation signals

- For every accepted PTM, site-determining ions must be found on at least one ion side on each side of the assigned position (left and right); assignments lacking either side should be rejected
- Reproduce the H3K27me2 reference output: 2 site-determining ions on the left side and 8 on the right side
- Output marks conform to standard Brno nomenclature (e.g. K4me3K14ac)
- The reported ion counts and types (c and z) match the spectrum's ETD fragments within the user-set MS/MS tolerance and spectrum-score filter
- The result file can be loaded as the Histone Coder result input by isoScale without format errors

## Limitations

- Requires ETD-type c/z fragment ion series; the tool is aimed at middle-down/top-down histone analyses, not generic bottom-up peptide data
- Validation depends on chosen spectrum score, fragment ion type and MS/MS tolerance filters; overly permissive tolerance can accept ambiguous localizations
- Only PTMs in the script are handled: phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K)

## Evidence

- [readme] Requirements: The software activates with Java. The input is Mascot output in .xml extension.: [evidence span withheld — anchor status: external_doc]
