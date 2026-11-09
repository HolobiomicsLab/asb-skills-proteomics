---
name: modification-site-mapping
description: Use when X!Tandem output XML files must be converted into per-peptide
  identifications with modification states, scores, and spectral support data for
  downstream tabulation or analysis.
when_to_use_negative:
- Search results come from MASCOT, OMSSA, SEQUEST or VEMS rather than X!Tandem XML
  — parser libraries like Mascot-Dat File and OMSSA Parser exist for those formats
- The goal is search-engine-independent visualization and analysis of peptide/protein
  identifications from multiple engines — the README recommends PeptideShaker for
  that
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0601
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into the object model (XTandemFile,
    Spectrum, PeptideMap, ModificationMap, InputParams, SupportData) from which modification
    and identification details are extracted
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Implementation language; the parser is consumed as a Java (Maven) dependency
    in analysis scripts
  repo: https://github.com/compomics/xtandem-parser
- name: X!Tandem
  role: Open-source search engine whose XML output files are the raw input to parse
- name: XTandem Viewer
  role: Lightweight GUI built on top of the parser for visual inspection of parsed
    results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:aa61fafb58ac586e60cda26b5bd20306a1ea25428fb43cf7c48e00ce9a916066
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - X!Tandem
  - XTandem Parser
  - XTandem Viewer
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# modification-site-mapping

## Summary

Extracts per-peptide fixed and variable modification assignments from X!Tandem XML search results by parsing them into the XTandem Parser Java object model, alongside sequence, expectation value (E-value) and hyperscore per identification.

## When to use

Use when X!Tandem output XML files must be converted into per-peptide identifications with modification states, scores, and spectral support data for downstream tabulation or analysis.

## When NOT to use

- Search results come from MASCOT, OMSSA, SEQUEST or VEMS rather than X!Tandem XML — parser libraries like Mascot-Dat File and OMSSA Parser exist for those formats
- The goal is search-engine-independent visualization and analysis of peptide/protein identifications from multiple engines — the README recommends PeptideShaker for that

## Inputs

- X!Tandem output XML result file (format documented at thegpm.org/docs/X_series_output_form.pdf)
- Optional: X!Tandem input XML file (input format documented at thegpm.org/tandem/api/index.html)

## Outputs

- Peptide objects (via PeptideMap) carrying identified sequence, expectation value (E-value) and X!Tandem hyperscore
- Fixed and variable modification assignments per identification (via ModificationMap)
- Search input settings (InputParams object)
- Fragment-ion m/z and intensity values per spectrum (SupportData instances)
- Tab-delimited identifications table (peptide_identifications.tsv)

## How to apply

Instantiate the XTandem Parser XTandemFile object with the X!Tandem XML result file as the sole input; the library parses the XML into an easily accessible Java object model. Iterate over all Spectrum objects and, via the PeptideMap, retrieve each Peptide's identified sequence, expectation value (E-value) and X!Tandem hyperscore. Read the fixed and variable modifications for each identification from the ModificationMap, and the search input settings from the InputParams object. Optionally pull fragment-ion m/z and intensity values from each spectrum's SupportData instance. Finally write all identifications to a tab-delimited file (e.g. peptide_identifications.tsv) that can be opened as a spreadsheet.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into the object model (XTandemFile, Spectrum, PeptideMap, ModificationMap, InputParams, SupportData) from which modification and identification details are extracted) — http://xtandem-parser.googlecode.com
- **Java** (Implementation language; the parser is consumed as a Java (Maven) dependency in analysis scripts) — https://github.com/compomics/xtandem-parser
- **X!Tandem** (Open-source search engine whose XML output files are the raw input to parse)
- **XTandem Viewer** (Lightweight GUI built on top of the parser for visual inspection of parsed results)

## Examples

```
XTandemFile xtf = new XTandemFile("tandem_output.xml"); // then iterate spectra: for each spectrum, read PeptideMap (sequence, E-value, hyperscore) and ModificationMap (fixed/variable modifications)
```

## Evaluation signals

- Every extracted row contains non-empty sequence, expectation value (E-value) and hyperscore fields
- Fixed and variable modifications are non-empty/readable for each identification via the ModificationMap
- Parsing completes within the paper's performance envelope: all tests run in less than 1 GB of RAM
- Parsing time consistent with reported benchmarks (e.g. 3225 quality spectra with 101 valid models in <1 s; 223327 spectra with 7644 models in 10 s)
- Output tab-delimited file opens as a spreadsheet with one row per identification and columns for sequence, E-value, hyperscore and modifications

## Limitations

- Parser is specific to the X!Tandem XML output format; other engines' formats (MASCOT .dat, OMSSA) require different parsers
- The parser is fast and lightweight but its published performance envelope assumes runs under 1 GB of RAM
- No discussion section content available in the source document, so caveats beyond the abstract/intro are not documented

## Evidence

- [abstract] The XTandem Parser software also includes the XTandem Viewer, a lightweight graphical user interface built on top of the parser: [evidence span withheld — anchor status: not_found]
