---
name: proteomics-identification-data-handling
description: Use when X!Tandem output XML files (per thegpm.org X_series output format)
  need to be parsed into an accessible object model — e.g. extracting identifications,
  valid models, or spectrum-level results for downstream proteomics analysis.
when_to_use_negative:
- Input files are raw instrument data or spectra files that have not yet been searched
  — run X!Tandem (e.g. via SearchGUI) first; this skill only handles the search output
  XML
- Results are from other search engines such as MASCOT, OMSSA, SEQUEST, or VEMS —
  use Mascot-Dat File or OMSSA Parser for those formats
- A search-engine-independent GUI analysis of already-parsed results is the goal —
  use PeptideShaker directly instead of building on the parser
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: XTandem Parser
  role: Java library that parses X!Tandem XML result files into an easily accessible
    and fully functional object model; also includes the XTandem Viewer GUI
  repo: github.com/compomics/xtandem-parser
- name: XTandem Viewer
  role: Lightweight graphical user interface built on top of the parser for visualizing
    parsed X!Tandem results
  repo: github.com/compomics/xtandem-parser
- name: X!Tandem
  role: Open-source search engine that produces the XML result files consumed by the
    parser
- name: PeptideShaker
  role: Recommended search-engine-independent platform for visualization and analysis
    of X!Tandem results after parsing
- name: Java
  role: Implementation language of the parser; used to instantiate XTandemFile objects
    and read identifications
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:0a792d9932b5e7a44527f4f5990207573ed8e76699d547059ac54460d93023f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - PeptideShaker
  - X!Tandem
  - XTandem Parser
  - XTandem Viewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# proteomics-identification-data-handling

## Summary

Parse and programmatically access X!Tandem XML MS/MS search result files using the open-source Java XTandem Parser library, converting search engine output into a fully functional object model for protein identification analysis, with benchmark parsing times (<1 s to 10 s) and peak memory below 1 GB.

## When to use

Use when X!Tandem output XML files (per thegpm.org X_series output format) need to be parsed into an accessible object model — e.g. extracting identifications, valid models, or spectrum-level results for downstream proteomics analysis.

## When NOT to use

- Input files are raw instrument data or spectra files that have not yet been searched — run X!Tandem (e.g. via SearchGUI) first; this skill only handles the search output XML
- Results are from other search engines such as MASCOT, OMSSA, SEQUEST, or VEMS — use Mascot-Dat File or OMSSA Parser for those formats
- A search-engine-independent GUI analysis of already-parsed results is the goal — use PeptideShaker directly instead of building on the parser

## Inputs

- X!Tandem output XML result files (format documented at http://www.thegpm.org/docs/X_series_output_form.pdf)
- Test dataset of X!Tandem XML files with 3225, 60902, and 223327 quality spectra distributed with the library

## Outputs

- XTandemFile Java object model exposing spectrum-level identifications, peptides, and valid models (e.g. 101, 2065, 7644 valid models for the benchmark files)
- Performance metrics (parsing time in seconds, peak RAM) if benchmarking, e.g. a performance_metrics.csv

## How to apply

Obtain the XTandem Parser library (Apache2 licence) from xtandem-parser.googlecode.com or the CompOmics GitHub repository, or add it as a Maven dependency (groupId de.proteinms.xtandemparser, artifactId xtandem-parser) from the Genesis maven2 repository. In a Java script, instantiate an XTandemFile object on each X!Tandem XML result file and read through all identifications to extract peptide and protein information from the search engine output. Validate performance against the paper's Table 1 benchmarks: files with 3225 quality spectra (101 valid models) parse in less than 1 second, 60902 spectra (2065 valid models) in 5 seconds, and 223327 spectra (7644 valid models) in 10 seconds, with all runs staying below 1 GB of RAM. For visualization or search-engine-independent analysis of the parsed results, the article recommends exporting to PeptideShaker rather than reimplementing viewers.

## Related tools

- **XTandem Parser** (Java library that parses X!Tandem XML result files into an easily accessible and fully functional object model; also includes the XTandem Viewer GUI) — github.com/compomics/xtandem-parser
- **XTandem Viewer** (Lightweight graphical user interface built on top of the parser for visualizing parsed X!Tandem results) — github.com/compomics/xtandem-parser
- **X!Tandem** (Open-source search engine that produces the XML result files consumed by the parser)
- **PeptideShaker** (Recommended search-engine-independent platform for visualization and analysis of X!Tandem results after parsing)
- **Java** (Implementation language of the parser; used to instantiate XTandemFile objects and read identifications)

## Examples

```
<dependency><groupId>de.proteinms.xtandemparser</groupId><artifactId>xtandem-parser</artifactId><version>1.13.0</version></dependency> then new XTandemFile("xtandem_output.xml")
```

## Evaluation signals

- Parsing time for the 3225-spectrum file (101 valid models) is under 1 second; 60902 spectra (2065 models) ~5 s; 223327 spectra (7644 models) ~10 s, matching Table 1
- Peak memory usage remains below 1 GB of RAM for all parsing runs, including the largest test file
- The number of valid models extracted from each file matches the reference counts (101, 2065, 7644) for the corresponding spectrum counts
- The instantiated XTandemFile object exposes all identifications from the XML without parse errors, consistent with the X_series output format
- The Maven dependency resolves from the Genesis maven2 repository (de.proteinms.xtandemparser:xtandem-parser) and the library runs under a standard JVM

## Limitations

- Performance and memory figures are guaranteed only for the paper's benchmark scale (up to ~223327 quality spectra per file); larger files are not characterized
- The parser handles only X!Tandem output XML — it does not read MASCOT (.dat), OMSSA, SEQUEST, or VEMS result formats
- The original xtandem-parser.googlecode.com URL is legacy; the active source is the CompOmics GitHub repository and Genesis maven2 repository
- No discussion/changelog content was available in the source document, so caveats beyond the abstract and README are unverified

## Evidence
