---
name: xtandem-xml-object-model-access
description: Use when the input is an X!Tandem XML output file from tandem MS/MS searching
  and per-peptide details (sequence, expectation value, hyperscore, modifications,
  spectra) must be extracted programmatically in Java.
when_to_use_negative:
- Input is a raw or generic search-engine output other than an X!Tandem XML result
  file — parsers such as Mascot-Dat File or OMSSA Parser target MASCOT and OMSSA files
  instead
- The goal is interactive visualization or cross-search-engine analysis of results
  — the README recommends PeptideShaker (or the bundled XTandem Viewer GUI) rather
  than writing a parser-based script
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into the object model (XTandemFile,
    Spectrum, Peptide/PeptideMap, ModificationMap, InputParams, SupportData) used
    for extraction
  repo: http://xtandem-parser.googlecode.com
- name: XTandem Viewer
  role: Lightweight GUI built on top of the parser; alternative to scripting for inspecting
    parsed results
- name: Java
  role: Language in which the parser library and extraction scripts are written; available
    as a Maven dependency (de.proteinms.xtandemparser:xtandem-parser)
  repo: https://github.com/compomics/xtandem-parser
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:910b2951781133dc570a62d29e9f7aa3b07bb16d8755f808513de165c8cff239
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - XTandem Parser
  - XTandem Viewer
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# xtandem-xml-object-model-access

## Summary

Use the XTandem Parser Java library to parse X!Tandem XML result files into a fully accessible object model, exposing per-spectrum peptide identifications (sequence, E-value, hyperscore), modifications, input parameters, and fragment-ion SupportData for downstream extraction and reporting.

## When to use

Use when the input is an X!Tandem XML output file from tandem MS/MS searching and per-peptide details (sequence, expectation value, hyperscore, modifications, spectra) must be extracted programmatically in Java.

## When NOT to use

- Input is a raw or generic search-engine output other than an X!Tandem XML result file — parsers such as Mascot-Dat File or OMSSA Parser target MASCOT and OMSSA files instead
- The goal is interactive visualization or cross-search-engine analysis of results — the README recommends PeptideShaker (or the bundled XTandem Viewer GUI) rather than writing a parser-based script

## Inputs

- X!Tandem XML result/output file (output format per thegpm.org X_series_output_form)
- X!Tandem input XML parameters (settings recorded in the file, exposed via InputParams)

## Outputs

- Peptide objects (identified sequence, expectation value/E-value, X!Tandem hyperscore) via the PeptideMap
- Modification data (fixed and variable modifications) via the ModificationMap
- InputParams object with search input settings
- SupportData instances with fragment-ion m/z and intensity values
- Tab-delimited file of all identifications (e.g., peptide_identifications.tsv)

## How to apply

Instantiate an XTandemFile object from the XTandem Parser library with the X!Tandem XML output file as the sole input. Iterate over the Spectrum objects and, via the PeptideMap, retrieve each Peptide's identified sequence, expectation value (E-value), and X!Tandem hyperscore. Read fixed and variable modifications from the ModificationMap and search settings from the InputParams object, then pull fragment-ion m/z and intensity values from each spectrum's SupportData instance. Write the extracted identifications to a tab-delimited file (e.g., peptide_identifications.tsv) so results can be opened as a spreadsheet. The library is designed to be fast and lightweight: parsing 3225 quality spectra with 101 valid models completes in under 1 s and 223327 spectra with 7644 models in 10 s, with all tests running in less than 1 GB of RAM.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into the object model (XTandemFile, Spectrum, Peptide/PeptideMap, ModificationMap, InputParams, SupportData) used for extraction) — http://xtandem-parser.googlecode.com
- **XTandem Viewer** (Lightweight GUI built on top of the parser; alternative to scripting for inspecting parsed results)
- **Java** (Language in which the parser library and extraction scripts are written; available as a Maven dependency (de.proteinms.xtandemparser:xtandem-parser)) — https://github.com/compomics/xtandem-parser

## Examples

```
XTandemFile xtandemFile = new XTandemFile("tandem_output.xml"); // then iterate Spectra, PeptideMap -> sequence, e-value, hyperscore
```

## Evaluation signals

- Every extracted row contains non-empty sequence, E-value, and hyperscore fields
- The parse-and-extract run completes within the reported performance envelope (all tests run in less than 1 GB of RAM; e.g., 223327 spectra with 7644 models parsed in 10 s)
- Number of parsed valid models matches the counts recorded in the X!Tandem XML (e.g., 101 valid models for 3225 quality spectra in the paper's benchmark)
- ModificationMap yields fixed and variable modifications consistent with the InputParams settings in the file
- Output tab-delimited file (peptide_identifications.tsv) opens cleanly as a spreadsheet with one row per identification

## Limitations

- All extracted content is limited to what X!Tandem recorded in its XML output format; no discussion section content was available in the source document
- Parsing performance depends on file size (e.g., ~223k spectra take ~10 s), and memory footprint should be watched against the sub-1 GB envelope
- For visualization and multi-search-engine analysis the library itself is not sufficient; the README recommends PeptideShaker for result analysis

## Evidence

- [other] other: [evidence span withheld — anchor status: not_found]
