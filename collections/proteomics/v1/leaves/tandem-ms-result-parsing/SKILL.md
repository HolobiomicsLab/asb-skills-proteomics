---
name: tandem-ms-result-parsing
description: Use when the input is an X!Tandem XML output file from a tandem MS search
  and you need to extract peptide sequences, E-values, hyperscores, modifications,
  or fragment-ion m/z and intensity values programmatically.
when_to_use_negative:
- The input is a Mascot .dat file or OMSSA result file — use Mascot-Dat File or OMSSA
  Parser instead
- You need search-engine-independent visualization/analysis across multiple search
  engines — PeptideShaker is recommended for that purpose
- You need to run the X!Tandem search itself rather than parse its output — use X!Tandem/SearchGUI
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into an accessible object
    model for extracting peptide identifications, scores, and spectra details
  repo: http://xtandem-parser.googlecode.com
- name: XTandem Viewer
  role: Lightweight GUI built on top of the parser for visualizing parsed X!Tandem
    results
  repo: http://xtandem-parser.googlecode.com
- name: X!Tandem
  role: Search engine whose XML output files are the target input for parsing
- name: Java
  role: Implementation language of the parser; Maven dependency groupId de.proteinms.xtandemparser
  repo: http://xtandem-parser.googlecode.com
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:9ae88fb8db964721614416142361dad41e771b6f602d7643e1556b7c416548ee
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
yes_rate: 0.6
n_claims: 10
---

# tandem-ms-result-parsing

## Summary

A skill for parsing X!Tandem XML result files into an accessible object model using the open-source Java XTandem Parser library, exposing per-spectrum peptide identifications, scores, modifications, and fragment-ion support data for downstream analysis.

## When to use

Use when the input is an X!Tandem XML output file from a tandem MS search and you need to extract peptide sequences, E-values, hyperscores, modifications, or fragment-ion m/z and intensity values programmatically.

## When NOT to use

- The input is a Mascot .dat file or OMSSA result file — use Mascot-Dat File or OMSSA Parser instead
- You need search-engine-independent visualization/analysis across multiple search engines — PeptideShaker is recommended for that purpose
- You need to run the X!Tandem search itself rather than parse its output — use X!Tandem/SearchGUI

## Inputs

- X!Tandem XML result/output file (format documented at thegpm.org/docs/X_series_output_form.pdf)
- X!Tandem input XML parameters file (format documented at thegpm.org/tandem/api)

## Outputs

- Java object model of parsed spectra (Spectrum, Peptide via PeptideMap)
- Per-peptide identified sequence, expectation value (E-value), and X!Tandem hyperscore
- Fixed and variable modifications per identification (ModificationMap)
- Fragment-ion m/z and intensity values from SupportData instances
- Tab-delimited peptide_identifications.tsv file of all identifications

## How to apply

Instantiate the XTandemFile object with the X!Tandem XML output file as the sole input, which parses the file into the library's Java object model. Iterate over the Spectrum objects and retrieve each Peptide via the PeptideMap, extracting the identified sequence, expectation value (E-value), and X!Tandem hyperscore for each identification. Read fixed and variable modifications from the ModificationMap and search settings from the InputParams object; pull fragment-ion m/z and intensity values from each spectrum's SupportData. Write the extracted identifications to a tab-delimited file (e.g. peptide_identifications.tsv) so results can be opened as a spreadsheet. Rely on the library's reported performance envelope — all tests run in less than 1 GB of RAM (e.g. 223327 spectra with 7644 valid models parsed in 10 s) — as a sanity check on the parse run.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into an accessible object model for extracting peptide identifications, scores, and spectra details) — http://xtandem-parser.googlecode.com
- **XTandem Viewer** (Lightweight GUI built on top of the parser for visualizing parsed X!Tandem results) — http://xtandem-parser.googlecode.com
- **X!Tandem** (Search engine whose XML output files are the target input for parsing)
- **Java** (Implementation language of the parser; Maven dependency groupId de.proteinms.xtandemparser) — http://xtandem-parser.googlecode.com

## Examples

```
XTandemFile xtf = new XTandemFile("tandem_output.xml"); // then iterate spectra, PeptideMap, ModificationMap, SupportData
```

## Evaluation signals

- Every extracted row contains non-empty peptide sequence, E-value, and hyperscore fields
- The parse-and-extract run completes within the reported performance envelope (all tests run in less than 1 GB of RAM)
- Parsing time matches reported scaling, e.g. 3225 quality spectra with 101 valid models parsed in about 1 s and 223327 spectra with 7644 models in 10 s
- ModificationMap and InputParams return fixed/variable modifications and input settings consistent with the X!Tandem input XML
- Output peptide_identifications.tsv is valid tab-delimited text openable as a spreadsheet

## Limitations

- Library is specific to X!Tandem XML output; it does not parse MASCOT (.dat) or OMSSA result files, for which Mascot-Dat File and OMSSA Parser exist
- No discussion-section content (changelog) was available in the source document, limiting documented caveats
- Fragment-ion details come only from each spectrum's SupportData, so peak-level richness depends on what X!Tandem serialized in the XML

## Evidence
