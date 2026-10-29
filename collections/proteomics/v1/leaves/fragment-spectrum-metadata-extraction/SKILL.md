---
name: fragment-spectrum-metadata-extraction
description: Use when you have X!Tandem XML result files and need to export detailed
  per-spectrum identification fields (peptide, E-value, hyperscore, protein accession,
  spectrum details) into a spreadsheet-openable format.
when_to_use_negative:
- Input files are MASCOT or OMSSA result files — use Mascot-Dat File or OMSSA Parser
  instead, since XTandem Parser is specific to X!Tandem XML output.
- You need interactive GUI-based visualization/analysis of X!Tandem results rather
  than scripted bulk export — the authors recommend PeptideShaker as a search-engine
  independent platform for visualization and analysis.
- You only need to view a single spectrum interactively — the bundled XTandem Viewer
  GUI (built on SwingX, JGoodies, and compomics-utilities) is designed for that rather
  than writing a parser script.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into an object model from
    which per-spectrum identification metadata is extracted
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Implementation language for the extraction script using the XTandem Parser
    API
- name: XTandem Viewer
  role: Lightweight GUI built on top of the parser for interactive inspection of the
    same identification data
  repo: http://xtandem-parser.googlecode.com
- name: X!Tandem
  role: Search engine that produced the XML result files being parsed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:37a7b9546d963e1f7bf671388dab64090edd626cdd5d349f579c6632d70ebeb2
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fragment-spectrum-metadata-extraction

## Summary

Extracts detailed per-spectrum identification metadata (peptide sequence, E-value, hyperscore, protein accession, charge, precursor m/z, fragment ion m/z/intensity) from X!Tandem XML output files into a tab-delimited table using the XTandem Parser Java library.

## When to use

Use when you have X!Tandem XML result files and need to export detailed per-spectrum identification fields (peptide, E-value, hyperscore, protein accession, spectrum details) into a spreadsheet-openable format.

## When NOT to use

- Input files are MASCOT or OMSSA result files — use Mascot-Dat File or OMSSA Parser instead, since XTandem Parser is specific to X!Tandem XML output.
- You need interactive GUI-based visualization/analysis of X!Tandem results rather than scripted bulk export — the authors recommend PeptideShaker as a search-engine independent platform for visualization and analysis.
- You only need to view a single spectrum interactively — the bundled XTandem Viewer GUI (built on SwingX, JGoodies, and compomics-utilities) is designed for that rather than writing a parser script.

## Inputs

- X!Tandem XML output result file (format documented at thegpm.org/docs/X_series_output_form.pdf)
- xtandem-parser Java library dependency (de.proteinms.xtandemparser:xtandem-parser)

## Outputs

- xtandem_identifications.tsv — a tab-delimited file with one row per identification, containing peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, spectrum filename, charge, precursor m/z, and fragment ion m/z/intensity

## How to apply

Load the X!Tandem XML output file with the XTandem File object of the XTandem Parser Java library, which parses the XML into an object model. Retrieve all fragmentation spectra as a collection of Spectrum objects and iterate over their identifications via the Peptide Map. For each identification, extract the peptide sequence, modified sequence, E-value (expectation value), X!Tandem hyperscore, protein accession number, plus spectrum details (spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity from Support Data). Write one identification per row to xtandem_identifications.tsv with tab-separated columns loadable as a spreadsheet. The parser is designed to stay lightweight: the paper reports all parsing tests run in less than 1 GB of RAM, with 223327 spectra (7644 valid models) parsed in 10 s.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into an object model from which per-spectrum identification metadata is extracted) — http://xtandem-parser.googlecode.com
- **Java** (Implementation language for the extraction script using the XTandem Parser API)
- **XTandem Viewer** (Lightweight GUI built on top of the parser for interactive inspection of the same identification data) — http://xtandem-parser.googlecode.com
- **X!Tandem** (Search engine that produced the XML result files being parsed)

## Examples

```
<dependency><groupId>de.proteinms.xtandemparser</groupId><artifactId>xtandem-parser</artifactId><version>X.Y.Z</version></dependency>
```

## Evaluation signals

- xtandem_identifications.tsv exists and is tab-delimited with exactly one row per identification in the input X!Tandem XML file.
- Each row contains all expected fields: peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity from Support Data.
- Parsing stays within the paper's reported performance envelope: <1 GB RAM and ~10 s for 223327 spectra with 7644 valid models.
- The output file opens correctly in a spreadsheet application given its tab-separated column layout.
- Extracted E-values and hyperscores for spot-checked spectra match the values present in the source X!Tandem XML.

## Limitations

- XTandem Parser is specific to X!Tandem XML output files and does not parse result files from other search engines such as MASCOT, OMSSA, SEQUEST, or VEMS.
- The repository's release wiki notes '_No changelog found._', so changelog information may not be available for tracking parser version changes.
- For visualization and analysis of results across multiple search engines, the authors direct users to PeptideShaker rather than building analysis on top of raw parser output.

## Evidence

- [abstract] we present an open-source Java library called XTandem Parser that parses X!Tandem XML result files into an easily accessible and fully functional object model: "we present an open-source Java library called XTandem Parser that parses X!Tandem XML result files into an easily accessible and fully functional object model"
- [abstract] All these tests run in less than 1GB of RAM.: "All these tests run in less than 1GB of RAM."
- [abstract] 3225 101 o1 60902 2065 5 223327 7644 10: "Number of quality spectra Number of valid models Parsing time (s) 3225 101 o1 60902 2065 5 223327 7644 10"
- [readme] XTandem Parser is a Java project for extracting information from X!Tandem output xml files.: "**XTandem Parser** is a Java project for extracting information from X!Tandem output xml files."
- [readme] To visualize and analyze X!Tandem results we recommend the use of [PeptideShaker](http://compomics.github.io/projects/peptide-shaker.html).: "To visualize and analyze X!Tandem results we recommend the use of [PeptideShaker](http://compomics.github.io/projects/peptide-shaker.html)."
