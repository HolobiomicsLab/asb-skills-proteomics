---
name: xtandem-xml-result-parsing
description: Use when you have X!Tandem XML output files from tandem MS/MS protein
  identification searches and need to programmatically extract detailed per-spectrum
  identification information, e.g. into a tab-delimited table.
when_to_use_negative:
- Input is a MASCOT (Mascot-Dat File) or OMSSA result file — those parsers (Mascot-Dat
  File, OMSSA Parser) are the appropriate tools instead
- The goal is visualization or search-engine-independent analysis of results from
  multiple engines — PeptideShaker is recommended instead of writing a custom parser
  script
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into an object model from
    which per-spectrum identifications and support data are extracted
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Language in which the XTandem Parser library and the export script are implemented
- name: X!Tandem
  role: Upstream search engine that produced the XML result files consumed by this
    skill
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:5e3d58eee1ba45b763c80b1bd8c63fad6184bac46ea46922159283ec5e0c10d2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - X!Tandem
  - XTandem Parser
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# xtandem-xml-result-parsing

## Summary

A method for parsing X!Tandem XML result files into an accessible Java object model with XTandem Parser, and exporting per-spectrum identification details (peptide sequence, E-value, hyperscore, protein accession, spectrum support data) to a tab-delimited spreadsheet-openable file.

## When to use

Use when you have X!Tandem XML output files from tandem MS/MS protein identification searches and need to programmatically extract detailed per-spectrum identification information, e.g. into a tab-delimited table.

## When NOT to use

- Input is a MASCOT (Mascot-Dat File) or OMSSA result file — those parsers (Mascot-Dat File, OMSSA Parser) are the appropriate tools instead
- The goal is visualization or search-engine-independent analysis of results from multiple engines — PeptideShaker is recommended instead of writing a custom parser script

## Inputs

- X!Tandem output XML file (format documented at http://www.thegpm.org/docs/X_series_output_form.pdf)

## Outputs

- xtandem_identifications.tsv — tab-delimited file with one row per identification, containing peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, spectrum filename, charge, precursor m/z, and fragment ion m/z/intensity

## How to apply

Load the X!Tandem XML output file with the XTandemFile object of the XTandem Parser Java library (available via Maven as de.proteinms.xtandemparser:xtandem-parser), which handles XML parsing into the object model. Retrieve all fragmentation spectra as a collection of Spectrum objects and iterate over their identifications via the Peptide Map. For each identification, extract the peptide sequence, modified sequence, E-value (expectation value), X!Tandem hyperscore, protein accession number, plus spectrum details: spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity from the Support Data. Write one identification per row to xtandem_identifications.tsv with tab-separated columns loadable as a spreadsheet. The parser is designed to be fast and lightweight — the paper reports 223327 spectra with 7644 valid models parsed in ~10 s, all tests under 1 GB RAM.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into an object model from which per-spectrum identifications and support data are extracted) — http://xtandem-parser.googlecode.com
- **Java** (Language in which the XTandem Parser library and the export script are implemented)
- **X!Tandem** (Upstream search engine that produced the XML result files consumed by this skill)

## Examples

```
XTandemFile xtandemFile = new XTandemFile(inputXmlFile); for (Spectrum spectrum : xtandemFile.getSpectra()) { /* iterate Peptide Map, extract E-value, hyperscore, accession, support data */ }
```

## Evaluation signals

- xtandem_identifications.tsv exists and is tab-delimited with exactly one row per identification in the input XML
- Each row contains all expected fields: peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity
- Parsing memory stays below the paper's reported envelope of <1 GB RAM
- Parsing time is consistent with the paper's benchmarks (e.g. 223327 spectra with 7644 valid models in ~10 s; 3225 quality spectra with 101 valid models in <1 s)
- The file opens correctly as a spreadsheet with columns separated by tabs

## Limitations

- The skill is specific to X!Tandem XML output; it does not parse other search engines' result formats
- The XML output format is defined by the X!Tandem project (X_series_output_form.pdf), so format changes upstream could break parsing
- Discussion-section content was not available in the source document, so additional caveats from the authors' discussion are untraced

## Evidence
