---
name: brno-nomenclature-ptm-annotation
description: Use when Mascot .xml results of histone peptide searches (phosphorylation
  S/T/Y, acetylation K, mono-/di-methylation K/R, trimethylation K) need standardized
  Brno-style PTM labels before abundance quantification.
when_to_use_negative:
- Input is Mascot .csv output rather than .xml — isoScale consumes the .csv, but Histone
  Coder's Brno conversion requires the .xml Mascot output.
- The goal is quantification (total ion intensity or FIRR of isobaric peptides) rather
  than PTM localization annotation and nomenclature conversion — that is isoScale's
  role.
- PTMs outside Histone Coder's supported set (phosphorylation S,T,Y; acetylation K;
  mono-/di-methylation K,R; trimethylation K) are involved.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Histone Coder
  role: Java GUI tool that counts MS/MS site-determining ions for PTM localization
    validation, applies score/fragment-ion/tolerance filters, and converts Mascot
    .xml output into Brno nomenclature; its result file feeds isoScale.
  repo: https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- name: Mascot
  role: Database search engine whose .xml output supplies the raw peptide-spectrum
    assignments that Histone Coder converts to Brno nomenclature.
- name: isoScale
  role: Downstream consumer of the Histone Coder result file; performs total-ion-intensity
    and isobaric-peptide (FIRR) quantification on the Brno-annotated identifications.
  repo: https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201400084
    title: Middle‐down hybrid chromatography/tandem mass spectrometry workflow for
      characterization of combinatorial post‐translational modifications in histones
  source_content_hash: sha256:9a8deeb705a56da2569af0a9d94d792b189aed439a148ba37c2144303a591910
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Histone Coder
  - Iso Scale
  - Java
  - Mascot
  - isoScale
  - pandas
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# brno-nomenclature-ptm-annotation

## Summary

Convert Mascot PTM assignments on histone MS/MS identifications into the standard Brno nomenclature (e.g. K4me3K14ac), so that combinatorial histone marks from middle-down/top-down experiments are represented consistently for downstream quantification with isoScale.

## When to use

Use when Mascot .xml results of histone peptide searches (phosphorylation S/T/Y, acetylation K, mono-/di-methylation K/R, trimethylation K) need standardized Brno-style PTM labels before abundance quantification.

## When NOT to use

- Input is Mascot .csv output rather than .xml — isoScale consumes the .csv, but Histone Coder's Brno conversion requires the .xml Mascot output.
- The goal is quantification (total ion intensity or FIRR of isobaric peptides) rather than PTM localization annotation and nomenclature conversion — that is isoScale's role.
- PTMs outside Histone Coder's supported set (phosphorylation S,T,Y; acetylation K; mono-/di-methylation K,R; trimethylation K) are involved.

## Inputs

- Mascot search engine output in .xml extension (identified histone peptide spectra)
- User-selected filters: spectrum score, fragment ion types, MS/MS tolerance

## Outputs

- Histone Coder result file with PTMs annotated in standard Brno nomenclature (e.g. K4me3K14ac) and listed site-determining ion counts per spectrum
- Result file formatted for direct input into isoScale for total-ion-intensity quantification

## How to apply

Run Histone Coder (a Java tool) on the Mascot output in .xml extension; it parses each identified histone spectrum, counts the number of MS/MS site-determining ions between the Mascot-assigned PTM localization and the closest other amino acids that can host the modification, and lists the number and type of these ions. Via the interface, select filters for spectrum score, fragment ion types, and MS/MS tolerance before conversion. The tool then rewrites the Mascot-assigned modifications using the standard Brno nomenclature (e.g. K4me3K14ac), producing a result file that serves as one of the two inputs (together with the Mascot .csv output) for isoScale quantification of single and co-fragmented isobaric polypeptide species.

## Related tools

- **Histone Coder** (Java GUI tool that counts MS/MS site-determining ions for PTM localization validation, applies score/fragment-ion/tolerance filters, and converts Mascot .xml output into Brno nomenclature; its result file feeds isoScale.) — https://github.com/middle-down/Software/blob/master/Histone%20Coder.jar?raw=true
- **Mascot** (Database search engine whose .xml output supplies the raw peptide-spectrum assignments that Histone Coder converts to Brno nomenclature.)
- **isoScale** (Downstream consumer of the Histone Coder result file; performs total-ion-intensity and isobaric-peptide (FIRR) quantification on the Brno-annotated identifications.) — https://github.com/middle-down/Software/blob/master/isoScale-v2.jar?raw=true

## Examples

```
java -jar Histone\ Coder.jar  # load Mascot .xml output in the GUI, set spectrum score, fragment ion type and MS/MS tolerance filters, then export the Brno-nomenclature result file for isoScale
```

## Evaluation signals

- Each annotated histone peptide carries a Brno-style modification string (e.g. K4me3K14ac) rather than Mascot-native PTM labels.
- For every localized PTM, the output lists the number and type of site-determining ions between the assigned localization and the closest alternative host residues; zero or missing ion evidence flags ambiguous localization.
- The Histone Coder result file, combined with the Mascot .csv data file, is accepted by isoScale without format errors.
- Filter settings (spectrum score, fragment ion types, MS/MS tolerance) selected in the interface are consistent with the fragmentation mode (CID/HCD vs ETD) and resolution used downstream.

## Limitations

- Histone Coder supports only phosphorylation (S,T,Y), acetylation (K), mono-/dimethylation (K,R), and trimethylation (K); other PTMs are not converted.
- The tool is a Java GUI application and requires Mascot output in .xml extension; it cannot be scripted directly from a .csv search result.
- PTM localization confidence depends on the counted site-determining ions and the user-chosen filters; ambiguous spectra with insufficient site-determining ions remain uncertain after conversion.
- Requires Java runtime.

## Evidence

- [readme] Histone Coder converts Mascot .xml output to standard Brno nomenclature for PTMs: "As additional feature, Histone Coder converts the Mascot output to the standard Brno nomenclature for PTMs (e.g. K4me3K14ac)."
- [intro] Histone Coder counts site-determining MS/MS ions to validate PTM localization: "Histone Coder counts the number of MS/MS ions in a given spectrum to determine the unambiguous localization of a post-translational modification (PTM)."
- [readme] Supported PTM types and interface filters: "The PTMs included in the script are phosphorylation (S,T,Y), acetylation (K), mono- and dimethylation (K,R) and trimethylation (K)."
- [readme] Input requirements: Java, Mascot .xml: [evidence span withheld — anchor status: external_doc]
- [methods] Result file feeds isoScale quantification: "The software input is the result file, which is the output format of Histone Coder, and a data file, which is the Mascot .csv output"
- [intro] Histone Coder and isoScale produced to validate and quantify Mascot identifications: "The page currently contains Histone Coder and isoScale, two software produced to validate MS/MS spectra and quantify identified polypeptides by Mascot (Matrix Science, UK) database searching engine."
