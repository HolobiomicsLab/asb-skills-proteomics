---
name: fragment-ion-mz-intensity-extraction
description: Use when you have an X!Tandem XML output file and need the underlying
  fragment-ion m/z and intensity values for each parsed spectrum, e.g. for downstream
  spectral inspection or identification detail tables.
when_to_use_negative:
- Input is not X!Tandem XML output (e.g. MASCOT .dat or OMSSA result files) — those
  have their own parsers (Mascot-Dat File, OMSSA Parser)
- The goal is cross-search-engine visualization/analysis of results — use PeptideShaker,
  the search-engine-independent platform the authors recommend
- Only search input parameters or modification settings are needed without ion-level
  data — the InputParams and ModificationMap objects suffice without touching SupportData
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library that parses the X!Tandem XML result file and exposes SupportData
    instances holding fragment-ion m/z and intensity values per spectrum
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Implementation language of the parser; ion extraction is performed as Java
    calls on the XTandemFile/Spectrum/SupportData object model
- name: XTandem Viewer
  role: GUI built on top of the parser that visualizes the spectra/ions accessible
    through the same object model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:727542d3ded31129da1c893f45179f99b80341daf16bff2fa689780255f12efc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - XTandem Parser
  - XTandem Viewer
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# fragment-ion-mz-intensity-extraction

## Summary

Extract per-spectrum fragment-ion m/z and intensity values from the SupportData instances exposed by the XTandem Parser Java object model after parsing an X!Tandem XML result file.

## When to use

Use when you have an X!Tandem XML output file and need the underlying fragment-ion m/z and intensity values for each parsed spectrum, e.g. for downstream spectral inspection or identification detail tables.

## When NOT to use

- Input is not X!Tandem XML output (e.g. MASCOT .dat or OMSSA result files) — those have their own parsers (Mascot-Dat File, OMSSA Parser)
- The goal is cross-search-engine visualization/analysis of results — use PeptideShaker, the search-engine-independent platform the authors recommend
- Only search input parameters or modification settings are needed without ion-level data — the InputParams and ModificationMap objects suffice without touching SupportData

## Inputs

- X!Tandem XML result file (output xml format from the X!Tandem search engine; example test file provided as a zip in the xtandem-parser downloads)
- Java project with the xtandem-parser Maven dependency (de.proteinms.xtandemparser:xtandem-parser)

## Outputs

- Fragment-ion m/z and intensity values per spectrum, retrieved from SupportData instances
- Optionally, a tab-delimited detail file (e.g. peptide_identifications.tsv) joining ion data with peptide sequence, E-value and hyperscore per identification

## How to apply

Instantiate an XTandemFile object from the XTandem Parser library with the X!Tandem XML result file as the sole input; the parser converts the XML into an accessible Java object model. Iterate over the Spectrum objects and pull the fragment-ion m/z and intensity values from each spectrum's SupportData instances. For the same spectra, Peptide identifications (sequence, expectation value E-value, hyperscore) are available via the PeptideMap, and modifications via the ModificationMap, so ion data can be joined to identifications in a single pass. Write combined results (e.g. peptide_identifications.tsv) in a tab-delimited layout so the extracted detail can be opened as a spreadsheet. Rationale: the parser is designed to expose ion-level detail that the flat X!Tandem XML hides in deeply nested elements, while staying lightweight — all performance tests ran in less than 1GB of RAM.

## Related tools

- **XTandem Parser** (Java library that parses the X!Tandem XML result file and exposes SupportData instances holding fragment-ion m/z and intensity values per spectrum) — http://xtandem-parser.googlecode.com
- **Java** (Implementation language of the parser; ion extraction is performed as Java calls on the XTandemFile/Spectrum/SupportData object model)
- **XTandem Viewer** (GUI built on top of the parser that visualizes the spectra/ions accessible through the same object model)

## Examples

```
XTandemFile xtf = new XTandemFile("tandem_output.xml"); // iterate xtf.getSpectrumMap() and read each Spectrum's SupportData m/z and intensity values
```

## Evaluation signals

- Every extracted spectrum row has non-empty SupportData-derived m/z and intensity values, and any joined identification row has non-empty sequence, E-value and hyperscore fields
- The parse-and-extract run completes within the paper's performance envelope: all tests run in less than 1GB of RAM
- Number of spectra processed matches the XML (paper benchmarks: 3225 quality spectra with 101 valid models parsed in <1 s; 223327 spectra with 7644 models in 10 s)
- Tab-delimited output opens correctly as a spreadsheet with one row per identification/spectrum and consistent column schema
- m/z and intensity values match the values present in the source X!Tandem XML (thegpm.org X_series output format) for spot-checked spectra

## Limitations

- Scope is X!Tandem output XML only; other engines' formats (MASCOT, OMSSA, SEQUEST, VEMS) are out of scope for this parser
- No discussion-section caveats were available in the source document ('No changelog found.'), so additional edge-case caveats are undocumented
- For visualization/analysis across multiple search engines, the authors redirect users to PeptideShaker rather than the raw parser

## Evidence

- [abstract] Java library parsing X!Tandem XML into object model: "we present an open-source Java library called XTandem Parser that parses X!Tandem XML result files into an easily accessible and fully functional object model"
- [abstract] Parser memory envelope: "All these tests run in less than 1GB of RAM."
- [intro] Purpose of the project: "**XTandem Parser** is a Java project for extracting information from X!Tandem output xml files."
- [readme] Example test file availability: "If you need an example file to play with, one is provided as a zip file in the downloads section on the right."
