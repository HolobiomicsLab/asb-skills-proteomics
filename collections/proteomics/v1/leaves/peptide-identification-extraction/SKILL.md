---
name: peptide-identification-extraction
description: Use when the input is an X!Tandem XML output file and per-identification
  details (peptide sequence, expectation value, hyperscore, modifications, protein
  accession, spectrum charge/precursor m/z, fragment m/z and intensity) must be extracted
  programmatically.
when_to_use_negative:
- Input is a MASCOT .dat file or OMSSA result file — use Mascot-Dat File or OMSSA
  Parser instead, as XTandem Parser only handles X!Tandem output XML
- Goal is interactive visualization or search-engine-independent analysis of results
  from multiple search engines — use PeptideShaker (or the bundled XTandem Viewer)
  rather than writing an extraction script
- Input is raw MS/MS spectra or the search itself has not been run — X!Tandem must
  first match tandem mass spectra against an in silico digested sequence database
  to produce the output XML
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: XTandem Parser
  role: Java library that parses the X!Tandem XML output into the object model (Spectrum/Peptide/ModificationMap/SupportData)
    from which identifications are extracted
  repo: http://xtandem-parser.googlecode.com
- name: X!Tandem
  role: Search engine that produced the XML result file being parsed; matches tandem
    mass spectra with peptide sequences
- name: Java
  role: Language in which the extraction script is implemented; XTandem Parser is
    available as a Maven dependency (de.proteinms.xtandemparser:xtandem-parser)
  repo: https://github.com/compomics/xtandem-parser
- name: XTandem Viewer
  role: GUI built on top of the parser for manual inspection of the same X!Tandem
    results
  repo: http://xtandem-parser.googlecode.com
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:68d662d9134b5b6927f76f2dc5563737ffcac3b5d544eeb880882925a4cf0bde
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - X!Tandem
  - XTandem Parser
  - XTandem Viewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peptide-identification-extraction

## Summary

Extracts per-spectrum peptide identifications (sequence, E-value, X!Tandem hyperscore), modifications, and fragment-ion data from X!Tandem XML result files using the XTandem Parser Java object model, and exports them to a tab-delimited spreadsheet-openable file.

## When to use

Use when the input is an X!Tandem XML output file and per-identification details (peptide sequence, expectation value, hyperscore, modifications, protein accession, spectrum charge/precursor m/z, fragment m/z and intensity) must be extracted programmatically.

## When NOT to use

- Input is a MASCOT .dat file or OMSSA result file — use Mascot-Dat File or OMSSA Parser instead, as XTandem Parser only handles X!Tandem output XML
- Goal is interactive visualization or search-engine-independent analysis of results from multiple search engines — use PeptideShaker (or the bundled XTandem Viewer) rather than writing an extraction script
- Input is raw MS/MS spectra or the search itself has not been run — X!Tandem must first match tandem mass spectra against an in silico digested sequence database to produce the output XML

## Inputs

- X!Tandem XML result (output) file
- X!Tandem input XML file (search settings referenced via InputParams)

## Outputs

- Tab-delimited TSV file (peptide_identifications.tsv / xtandem_identifications.tsv) with one row per identification: sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number/filename/charge, precursor m/z, fragment ion m/z and intensity
- In-memory XTandem Parser object model (Spectrum, Peptide, PeptideMap, ModificationMap, InputParams, SupportData)

## How to apply

Instantiate the XTandem Parser XTandemFile object with the X!Tandem XML output file as the sole input, which parses the XML into an accessible Java object model. Iterate over the Spectrum objects and, via the PeptideMap, retrieve each Peptide's identified sequence, expectation value (E-value) and X!Tandem hyperscore, plus protein accession and modified sequence. Read fixed and variable modifications from the ModificationMap and search settings from the InputParams object, and pull fragment-ion m/z and intensity values from each spectrum's SupportData instance. Write one identification per row to a tab-separated file (e.g. peptide_identifications.tsv or xtandem_identifications.tsv) loadable as a spreadsheet. Rationale: the library exists precisely to turn X!Tandem output XML into a structured object model, and the paper's usage pattern is a simple script reading all identifications and writing detailed results to a tab-delimited file.

## Related tools

- **XTandem Parser** (Java library that parses the X!Tandem XML output into the object model (Spectrum/Peptide/ModificationMap/SupportData) from which identifications are extracted) — http://xtandem-parser.googlecode.com
- **X!Tandem** (Search engine that produced the XML result file being parsed; matches tandem mass spectra with peptide sequences)
- **Java** (Language in which the extraction script is implemented; XTandem Parser is available as a Maven dependency (de.proteinms.xtandemparser:xtandem-parser)) — https://github.com/compomics/xtandem-parser
- **XTandem Viewer** (GUI built on top of the parser for manual inspection of the same X!Tandem results) — http://xtandem-parser.googlecode.com

## Examples

```
XTandemFile xFile = new XTandemFile("xtandem_output.xml"); // iterate Spectrum PeptideMap -> write sequence, e-value, hyperscore to peptide_identifications.tsv
```

## Evaluation signals

- Every extracted TSV row contains non-empty peptide sequence, E-value and hyperscore fields
- The parse-and-extract run stays within the paper's performance envelope: all tests run in less than 1 GB of RAM
- Parsing time scales as reported: 3225 quality spectra with 101 valid models in <1 s and 223327 spectra with 7644 valid models in ~10 s
- peptide_identifications.tsv exists, is tab-delimited, and has exactly one row per identification in the X!Tandem output file
- Fixed/variable modifications read from the ModificationMap are consistent with the modification annotations present in the source X!Tandem XML

## Limitations

- The library is specific to X!Tandem output XML; it does not parse MASCOT or OMSSA result files (dedicated parsers exist for those)
- For visualization and multi-engine analysis the README recommends PeptideShaker rather than custom extraction
- No discussion-section content was available in the source document, so edge-case failure modes beyond format scope are not documented

## Evidence
