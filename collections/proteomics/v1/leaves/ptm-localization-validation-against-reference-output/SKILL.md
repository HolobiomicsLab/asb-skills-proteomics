---
name: ptm-localization-validation-against-reference-output
description: Use when middle-down ETD MS/MS histone data have been searched with Mascot
  and PTM assignments (phospho S/T/Y, acetyl K, mono-/di-methyl K/R, tri-methyl K)
  must be proven unambiguous before quantification with isoScale.
when_to_use_negative:
- Input is a Mascot .csv output rather than the .xml format Histone Coder requires
  (the .csv is the isoScale data file instead)
- Data come from bottom-up peptide-centric analyses where Histone Coder is not recommended;
  the README recommends isoScale-style handling only for middle-down or top-down analyses
- No MS/MS spectra with site-determining c or z ions flank a candidate PTM, so localization
  cannot be validated and quantification would rest on ambiguous assignments
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: Histone Coder
  role: Java tool that counts MS/MS site-determining ions to validate Mascot PTM localization
    and converts output to Brno nomenclature
  repo: github.com/middle-down/Software
- name: Mascot
  role: Database-search engine whose .xml output supplies the initial PTM localizations
    being validated
- name: Java
  role: Runtime requirement to activate the Histone Coder software
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:0cdd3e0be5b730b5e80cdc7221912ad5385ea912d7d18f2c05eb283f76475670
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Java
  - Mascot
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ptm-localization-validation-against-reference-output

## Summary

Validates Mascot-assigned PTM localizations in middle-down histone MS/MS data by counting site-determining c/z fragment ions flanking each modification, and confirms the analysis by reproducing the published Histone Coder reference output (e.g. H3K27me2 with 2 left and 8 right site-determining ions).

## When to use

Use when middle-down ETD MS/MS histone data have been searched with Mascot and PTM assignments (phospho S/T/Y, acetyl K, mono-/di-methyl K/R, tri-methyl K) must be proven unambiguous before quantification with isoScale.

## When NOT to use

- Input is a Mascot .csv output rather than the .xml format Histone Coder requires (the .csv is the isoScale data file instead)
- Data come from bottom-up peptide-centric analyses where Histone Coder is not recommended; the README recommends isoScale-style handling only for middle-down or top-down analyses
- No MS/MS spectra with site-determining c or z ions flank a candidate PTM, so localization cannot be validated and quantification would rest on ambiguous assignments

## Inputs

- Mascot database-search output file in .xml extension containing peptide-spectrum matches with assigned PTM localizations
- User-selected interface settings: spectrum score filter, fragment ion types, and MS/MS ion tolerance

## Outputs

- Histone Coder result file listing each combinatorial histone mark with the number and type of site-determining ions found between the assigned PTM localization and the closest alternative host residues
- PTM localizations converted to standard Brno nomenclature (e.g. K4me3K14ac), suitable as input to isoScale

## How to apply

Load the Mascot database-search output in .xml format into the Java-based Histone Coder (Histone Coder.jar from the middle-down/Software repository). For each Mascot-assigned PTM, locate the nearest other amino acids in the histone sequence that can host the same modification, then count site-determining c and z ions on both sides of each alternative site, selecting filters for spectrum score, fragment ion types and MS/MS tolerance in the graphical interface. Convert filtered Mascot identifications to standard Brno nomenclature (e.g. K4me3K14ac) and write the result file listing the number and type of site-determining ions between the assigned PTM and the closest alternative host residues. Validate the run by comparing against the Fig 2B reference extract, where the H3K27me2 example reports 2 site-determining ions on the left side and 8 on the right side; a PTM is only accepted when at least one site-determining ion supports each side of the assigned position.

## Related tools

- **Histone Coder** (Java tool that counts MS/MS site-determining ions to validate Mascot PTM localization and converts output to Brno nomenclature) — github.com/middle-down/Software
- **Mascot** (Database-search engine whose .xml output supplies the initial PTM localizations being validated)
- **Java** (Runtime requirement to activate the Histone Coder software)

## Examples

```
java -jar "Histone Coder.jar"  # then in the GUI: load Mascot .xml output, set spectrum score / fragment ion types / MS/MS tolerance filters, and export the result file
```

## Evaluation signals

- Reproduce the Fig 2B reference output extract: the H3K27me2 example reports 2 site-determining ions on the left side and 8 on the right side
- Every accepted PTM has at least one site-determining ion on each side of the assigned PTM position
- Result file lists, for each combinatorial mark, the number and type of site-determining ions between the assigned localization and the closest alternative host residues
- Output PTMs follow standard Brno nomenclature (e.g. K4me3K14ac) rather than raw Mascot modification labels
- Filter settings (spectrum score, fragment ion types, MS/MS tolerance) are recorded from the interface for reproducibility

## Limitations

- Histone Coder only supports the PTM types built into the script: phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K)
- The input must be Mascot output in .xml extension; .csv Mascot output is only processed by isoScale downstream
- If a spectrum lacks site-determining ions on both sides of an alternative site, the PTM assignment cannot be validated and the mark is not accepted

## Evidence

- [other] Histone Coder validates Mascot PTM assignments by accepting only PTMs with at least one site-determining ion on each side of the assigned PTM position.: [evidence span withheld — anchor status: not_found]
