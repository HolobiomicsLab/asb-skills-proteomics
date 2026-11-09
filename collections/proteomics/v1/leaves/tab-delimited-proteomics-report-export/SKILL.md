---
name: tab-delimited-proteomics-report-export
description: Use when you have an X!Tandem output XML file and need each identification's
  peptide sequence, E-value, hyperscore, protein accession, and spectrum details (charge,
  precursor and fragment m/z) dumped as a flat table.
when_to_use_negative:
- The result files come from a different search engine (e.g. MASCOT, OMSSA, SEQUEST)
  — Mascot-Dat File and OMSSA Parser exist for those formats, not XTandem Parser
- The goal is interactive visualization or multi-search-engine comparison rather than
  a flat export — the article recommends PeptideShaker for search-engine-independent
  visualization and analysis
- Only a summary of results is needed and parsing the full XML object model would
  be unnecessary overhead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library whose XTandem File object parses the X!Tandem XML output into
    the object model from which identifications and spectrum support data are extracted
  repo: http://xtandem-parser.googlecode.com
- name: X!Tandem
  role: Search engine that produced the input XML result file by matching tandem mass
    spectra against theoretical ions from in silico digestion
- name: Java
  role: Implementation language for the export script built on the XTandem Parser
    library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:e0f5bedcaf509716fba426a94df4388d56dc78285ff0aa0b2287cd4eef5db027
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - X!Tandem
  - XTandem Parser
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tab-delimited-proteomics-report-export

## Summary

A scripted export of per-spectrum peptide identifications from an X!Tandem XML result file into a tab-delimited, spreadsheet-openable TSV, using the XTandem Parser Java object model. It preserves the paper's performance envelope (<1 GB RAM, ~10 s for 223327 spectra).

## When to use

Use when you have an X!Tandem output XML file and need each identification's peptide sequence, E-value, hyperscore, protein accession, and spectrum details (charge, precursor and fragment m/z) dumped as a flat table.

## When NOT to use

- The result files come from a different search engine (e.g. MASCOT, OMSSA, SEQUEST) — Mascot-Dat File and OMSSA Parser exist for those formats, not XTandem Parser
- The goal is interactive visualization or multi-search-engine comparison rather than a flat export — the article recommends PeptideShaker for search-engine-independent visualization and analysis
- Only a summary of results is needed and parsing the full XML object model would be unnecessary overhead

## Inputs

- X!Tandem output XML result file (format documented at thegpm.org/docs/X_series_output_form.pdf)

## Outputs

- xtandem_identifications.tsv — a tab-delimited file with one row per identification, openable as a spreadsheet

## How to apply

Load the X!Tandem XML output with the XTandem File object of the XTandem Parser Java library, which parses the result file into an accessible object model. Retrieve all fragmentation spectra as a collection of Spectrum objects and iterate their identifications via the Peptide Map. For each identification extract the peptide sequence, modified sequence, E-value (expectation value), X!Tandem hyperscore, and protein accession number, plus spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity from the Support Data. Write one identification per row to xtandem_identifications.tsv with tab-separated columns so the file loads directly in a spreadsheet.

## Related tools

- **XTandem Parser** (Java library whose XTandem File object parses the X!Tandem XML output into the object model from which identifications and spectrum support data are extracted) — http://xtandem-parser.googlecode.com
- **X!Tandem** (Search engine that produced the input XML result file by matching tandem mass spectra against theoretical ions from in silico digestion)
- **Java** (Implementation language for the export script built on the XTandem Parser library)

## Evaluation signals

- xtandem_identifications.tsv exists and is tab-delimited with exactly one row per identification
- Every extracted row contains peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity from Support Data
- Parsing stays under 1 GB of RAM (the paper reports all performance tests run in less than 1 GB)
- Parsing time scales as reported: 223327 spectra with 7644 valid models parse in about 10 s (and 3225 spectra with 101 models in ~1 s)

## Limitations

- The parser is specific to X!Tandem output XML; it does not read results from other search engines such as MASCOT, OMSSA, SEQUEST, or VEMS
- Memory and runtime grow with the number of quality spectra and valid models, though tests stayed under 1 GB of RAM
- For visualization and analysis beyond flat export, the README and article steer users to PeptideShaker (and SearchGUI for running searches) instead

## Evidence
