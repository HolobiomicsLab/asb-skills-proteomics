---
name: msms-search-score-interpretation
description: Use when input is an X!Tandem XML result file and you need per-spectrum
  identification details (peptide sequence, modified sequence, E-value, hyperscore,
  protein accession, charge, precursor and fragment m/z) exported for scoring interpretation.
when_to_use_negative:
- Input results are from MASCOT, OMSSA, SEQUEST or VEMS — XTandem Parser only handles
  X!Tandem XML output; use Mascot-Dat File or OMSSA Parser instead
- Goal is visualization and cross-search-engine analysis of identifications rather
  than scripted extraction — the paper recommends PeptideShaker, a search-engine independent
  platform, for that purpose
- Only a quick interactive inspection of spectra is needed — the bundled XTandem Viewer
  GUI built on top of the parser may suffice
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library that parses the X!Tandem XML result file into an object model
    (XTandem File, Spectrum, Peptide Map, Support Data) from which identification
    scores and spectrum details are extracted
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Language in which the extraction script and the XTandem Parser library are
    implemented
- name: X!Tandem
  role: Search engine whose XML output (quality spectra, valid models, hyperscores,
    E-values) is the parsed input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:f58fd41463f446c31860845147c2b5cf1212f00164c7666316ea3f68850f6bcc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - X!Tandem
  - XTandem Parser
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msms-search-score-interpretation

## Summary

Extracts detailed per-spectrum peptide identification results — E-value (expectation), X!Tandem hyperscore, protein accession, and spectrum properties — from X!Tandem XML output files using the open-source Java XTandem Parser library, which converts result files into an accessible object model.

## When to use

Use when input is an X!Tandem XML result file and you need per-spectrum identification details (peptide sequence, modified sequence, E-value, hyperscore, protein accession, charge, precursor and fragment m/z) exported for scoring interpretation.

## When NOT to use

- Input results are from MASCOT, OMSSA, SEQUEST or VEMS — XTandem Parser only handles X!Tandem XML output; use Mascot-Dat File or OMSSA Parser instead
- Goal is visualization and cross-search-engine analysis of identifications rather than scripted extraction — the paper recommends PeptideShaker, a search-engine independent platform, for that purpose
- Only a quick interactive inspection of spectra is needed — the bundled XTandem Viewer GUI built on top of the parser may suffice

## Inputs

- X!Tandem XML output file (format documented at thegpm.org X_series_output_form)
- Optional example X!Tandem XML file provided as a zip in the xtandem-parser downloads

## Outputs

- Tab-delimited spreadsheet-openable file (xtandem_identifications.tsv) with one row per identification: peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, filename, charge, precursor m/z, fragment ion m/z and intensity

## How to apply

Load the X!Tandem XML output file with the XTandem File object of the XTandem Parser Java library, which parses the XML into an object model. Retrieve all fragmentation spectra as a collection of Spectrum objects and iterate over their identifications via the Peptide Map. For each identification, extract the peptide sequence, modified sequence, E-value (expectation value), X!Tandem hyperscore, and protein accession number, plus spectrum details (spectrum number, filename, charge, precursor m/z, and fragment ion m/z/intensity from the Support Data). Write one identification per row to a tab-delimited file such as xtandem_identifications.tsv. Rationale: the paper validated the parser's speed and footprint — all performance tests ran in under 1 GB of RAM, with 223327 spectra and 7644 valid models parsed in 10 seconds — so scripted extraction at scale is feasible without special hardware.

## Related tools

- **XTandem Parser** (Java library that parses the X!Tandem XML result file into an object model (XTandem File, Spectrum, Peptide Map, Support Data) from which identification scores and spectrum details are extracted) — http://xtandem-parser.googlecode.com
- **Java** (Language in which the extraction script and the XTandem Parser library are implemented)
- **X!Tandem** (Search engine whose XML output (quality spectra, valid models, hyperscores, E-values) is the parsed input)

## Examples

```
XTandemFile xtandemFile = new XTandemFile("tandem_output.xml"); // iterate xtandemFile.getSpectra() and Peptide Map; write fields to xtandem_identifications.tsv
```

## Evaluation signals

- xtandem_identifications.tsv exists and is tab-delimited with exactly one row per identification
- Each row contains the expected fields: peptide sequence, modified sequence, E-value, hyperscore, protein accession, spectrum number, filename, charge, precursor m/z, fragment ion m/z/intensity
- Parsing stays within the paper's reported envelope: <1 GB of RAM, ~10 s for 223327 spectra with 7644 valid models (and <1 s for 3225 spectra with 101 models)
- Every Spectrum in the input XML yields a corresponding identification entry when iterating the Peptide Map
- E-values and hyperscores in the output match the values present in the source X!Tandem XML

## Limitations

- Only X!Tandem XML result files are supported; other engines (MASCOT, OMSSA, SEQUEST, VEMS) require their own parsers
- No discussion-section content is available in the source document, so caveats beyond reported performance are undocumented
- Original repository URL points to googlecode; the maintained source is at github.com/compomics/xtandem-parser with a ReleaseNotes wiki

## Evidence
