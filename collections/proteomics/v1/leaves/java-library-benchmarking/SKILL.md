---
name: java-library-benchmarking
description: Use when you must measure how fast and how memory-light a Java parser
  runs on XML result files of increasing size — e.g. X!Tandem output files with 3225,
  60902, or 233327 quality spectra — against a published reference.
when_to_use_negative:
- The goal is visualizing or biologically analyzing the X!Tandem identifications rather
  than benchmarking the parser (PeptideShaker is recommended for that).
- The input is not X!Tandem XML output — Mascot DAT files and OMSSA result files are
  handled by other parsers (Mascot-Dat File, OMSSA Parser).
- You need the X!Tandem search itself performed, not its output parsed; use the X!Tandem
  search engine (e.g. via SearchGUI) for searching.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: XTandem Parser
  role: 'Open-source Java library under benchmark: parses X!Tandem XML result files
    into an accessible object model via the XTandemFile object'
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Implementation language for the benchmark script; provides System.nanoTime
    timing and Runtime/JVM-profiler memory tracking
- name: X!Tandem
  role: Upstream search engine whose XML output files are the benchmark inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:b5696a82df60a4cf5bf5d4cbeb72fdcad089a6695f9326e87e60c2e03f7d2726
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - X!Tandem
  - XTandem Parser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# java-library-benchmarking

## Summary

Benchmark the wall-clock parsing time and peak memory of a Java library on files of graded sizes, as done for XTandem Parser on X!Tandem XML result files containing 3225, 60902 and 233327 quality spectra. Rationale: performance claims for bioinformatics parsers should be reproducible against published reference tables.

## When to use

Use when you must measure how fast and how memory-light a Java parser runs on XML result files of increasing size — e.g. X!Tandem output files with 3225, 60902, or 233327 quality spectra — against a published reference.

## When NOT to use

- The goal is visualizing or biologically analyzing the X!Tandem identifications rather than benchmarking the parser (PeptideShaker is recommended for that).
- The input is not X!Tandem XML output — Mascot DAT files and OMSSA result files are handled by other parsers (Mascot-Dat File, OMSSA Parser).
- You need the X!Tandem search itself performed, not its output parsed; use the X!Tandem search engine (e.g. via SearchGUI) for searching.

## Inputs

- X!Tandem XML output result files of graded size (3225, 60902, 233327 quality spectra; 101, 2065, 7644 valid models)
- XTandem Parser Java library (Apache2 licence) with its distributed test data set

## Outputs

- performance_metrics.csv with parsing time (s) and peak RAM per file
- Per-file counts of quality spectra and valid models for comparison with the reference table

## How to apply

Obtain the XTandem Parser library (Apache2 licence) and its distributed test set of X!Tandem XML output files, then write a small script that instantiates an XTandemFile object on each result file and reads through all identifications. Time each parsing run with a wall-clock timer such as Java's System.nanoTime, recording seconds per file. Track peak memory (e.g. Runtime memory tracking or a JVM profiler) to verify the reported constraint that all runs stay below 1 GB of RAM. Validate against the paper's Table 1 reference values: <1 s for 3225 quality spectra (101 valid models), ~5 s for 60902 spectra (2065 valid models), ~10 s for the largest file (7644 valid models), and compile times and peak RAM into a performance_metrics.csv.

## Related tools

- **XTandem Parser** (Open-source Java library under benchmark: parses X!Tandem XML result files into an accessible object model via the XTandemFile object) — http://xtandem-parser.googlecode.com
- **Java** (Implementation language for the benchmark script; provides System.nanoTime timing and Runtime/JVM-profiler memory tracking)
- **X!Tandem** (Upstream search engine whose XML output files are the benchmark inputs)

## Examples

```
XTandemFile xFile = new XTandemFile("tandem_output.xml"); xFile.load(); long t0 = System.nanoTime(); /* iterate all identifications, then compute elapsed seconds and peak RAM */
```

## Evaluation signals

- Parsing time for the 3225-spectrum file is below 1 second (reference: <1 s).
- Parsing time for the 60902-spectrum file is approximately 5 seconds and for the largest file approximately 10 seconds.
- Peak RAM remains below 1 GB for every parsing run.
- Counts of valid models parsed (101, 2065, 7644) match the Table 1 reference for each file.
- Timing uses wall-clock measurement (e.g. System.nanoTime) so results are comparable across runs.

## Limitations

- Reference times (<1 s / 5 s / 10 s) were reported in 2010 on the authors' hardware; absolute times will differ on modern machines, so only relative scaling and the <1 GB RAM constraint should be treated as invariants.
- The paper provides no discussion-section content and the repository shows 'No changelog found', so no additional caveats or failure modes are documented.
- The original googlecode URL is deprecated; the maintained source is now at github.com/compomics/xtandem-parser.

## Evidence

- [abstract] we present an open-source Java library called XTandem Parser that parses X!Tandem XML result files into an easily accessible and fully functional object model: "we present an open-source Java library called XTandem Parser that parses X!Tandem XML result files into an easily accessible and fully functional object model"
- [abstract] All these tests run in less than 1GB of RAM.: "All these tests run in less than 1GB of RAM."
- [abstract] Number of quality spectra Number of valid models Parsing time (s) 3225 101 o1 60902 2065 5 223327 7644 10: "Number of quality spectra Number of valid models Parsing time (s) 3225 101 o1 60902 2065 5 223327 7644 10"
- [readme] XTandem Parser is a Java project for extracting information from X!Tandem output xml files.: "XTandem Parser is a Java project for extracting information from X!Tandem output xml files."
