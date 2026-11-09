---
name: msms-search-result-file-parsing
description: Use when X!Tandem XML output files (e.g. from thegpm.org search engine)
  must be read programmatically to extract identifications, or when parsing-time/memory
  performance of search-result parsers is being benchmarked.
when_to_use_negative:
- Results come from other search engines (MASCOT, OMSSA, SEQUEST, VEMS) — use Mascot-Dat
  File or OMSSA Parser instead, or PeptideShaker for multi-engine analysis
- The goal is interactive visualization/analysis of X!Tandem results rather than programmatic
  parsing — use XTandem Viewer or PeptideShaker
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into an accessible object
    model; used to instantiate XTandemFile and iterate identifications
  repo: http://xtandem-parser.googlecode.com
- name: X!Tandem
  role: Open-source search engine whose XML output files are the parser's input
- name: Java
  role: Implementation language; scripts instantiate XTandemFile and time parsing
    runs
  repo: github.com/compomics/xtandem-parser
- name: XTandem Viewer
  role: GUI built on top of the parser for visualization of parsed results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:7b1fa25a67ed546dffbd25dcf867b2fec59bfcf137e5f9c00fc7b42d65ac4839
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# msms-search-result-file-parsing

## Summary

Parse X!Tandem MS/MS search result XML files into a Java object model using the XTandem Parser library, enabling programmatic extraction of peptide and protein identifications with fast, low-memory performance.

## When to use

Use when X!Tandem XML output files (e.g. from thegpm.org search engine) must be read programmatically to extract identifications, or when parsing-time/memory performance of search-result parsers is being benchmarked.

## When NOT to use

- Results come from other search engines (MASCOT, OMSSA, SEQUEST, VEMS) — use Mascot-Dat File or OMSSA Parser instead, or PeptideShaker for multi-engine analysis
- The goal is interactive visualization/analysis of X!Tandem results rather than programmatic parsing — use XTandem Viewer or PeptideShaker

## Inputs

- X!Tandem output XML result files (format documented at http://www.thegpm.org/docs/X_series_output_form.pdf)
- X!Tandem input XML files (format documented at http://www.thegpm.org/tandem/api/index.html)

## Outputs

- Java object model of identifications (peptide/protein matches) accessible via XTandemFile
- Performance metrics per file: parsing time (seconds) and peak RAM (e.g. compiled into performance_metrics.csv)

## How to apply

Obtain the XTandem Parser Java library (Apache2 licence, http://xtandem-parser.googlecode.com or github.com/compomics/xtandem-parser) or add it as a Maven dependency (groupId de.proteinms.xtandemparser). Write a script that instantiates an XTandemFile object on each X!Tandem XML result file and reads through all identifications, tracking the number of quality spectra (e.g. 3225, 60902, 233327) and valid models (101, 2065, 7644). Time each parsing run with a wall-clock timer (e.g. System.nanoTime) and monitor peak memory (Runtime tracking or a JVM profiler). Reference performance from Table 1: <1 s for 3225 spectra, 5 s for 60902 spectra, 10 s for 223327 spectra, with all runs staying below 1 GB RAM. Note the input and output XML formats are documented at thegpm.org, and an example file ships with the download.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into an accessible object model; used to instantiate XTandemFile and iterate identifications) — http://xtandem-parser.googlecode.com
- **X!Tandem** (Open-source search engine whose XML output files are the parser's input)
- **Java** (Implementation language; scripts instantiate XTandemFile and time parsing runs) — github.com/compomics/xtandem-parser
- **XTandem Viewer** (GUI built on top of the parser for visualization of parsed results)

## Examples

```
<dependency><groupId>de.proteinms.xtandemparser</groupId><artifactId>xtandem-parser</artifactId><version>1.13.0</version></dependency>
```

## Evaluation signals

- Parsing times approximate Table 1 reference values: <1 s for 3225 quality spectra (101 valid models), 5 s for 60902 spectra (2065 models), 10 s for 223327 spectra (7644 models)
- Peak RAM stays below 1 GB for all parsing runs
- Number of valid models parsed matches expected ratios to quality spectra for each file size
- Maven dependency resolves against the genesis.UGent.be / EBI repositories at version 1.13.0 (or latest)

## Limitations

- Performance figures are for X!Tandem XML files of specific sizes; other file sizes or formats are not benchmarked
- No discussion/changelog content available in source document; newer usage guidance points to PeptideShaker and SearchGUI rather than standalone XTandem Viewer
- Original download location (xtandem-parser.googlecode.com) may be superseded by the compomics GitHub repository
