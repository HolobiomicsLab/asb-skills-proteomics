---
name: parsing-performance-metrics-tabulation
description: Use when you need to quantify how quickly and with how much memory a
  Java parser reads X!Tandem XML output files at different scales (thousands to hundreds
  of thousands of quality spectra), and to compile the timing/RAM results into a metrics
  table.
when_to_use_negative:
- Input results come from other search engines (e.g. MASCOT .dat files or OMSSA result
  files) — use Mascot-Dat File or OMSSA Parser instead, since XTandem Parser only
  extracts information from X!Tandem output xml files.
- The goal is visualization or biological analysis of identifications rather than
  parser performance benchmarking — the paper recommends PeptideShaker as a search
  engine independent platform for that purpose.
- You need to parse spectra itself rather than an X!Tandem search output; this skill
  only times reading of already-produced X!Tandem XML result files.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: XTandem Parser
  role: Java library being benchmarked; instantiates XTandemFile objects to parse
    X!Tandem XML result files while timing and memory use are recorded
  repo: github.com/compomics/xtandem-parser
- name: Java
  role: Implementation language for the benchmark script (System.nanoTime timing,
    Runtime memory tracking)
- name: X!Tandem
  role: Upstream open-source search engine whose XML output files are the benchmark
    inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:c22a0e6d5e8f9f57ededed22e1be050a5d2ad7319a5452701ec9e8b1d267e4ff
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# parsing-performance-metrics-tabulation

## Summary

Benchmarks the wall-clock parsing time and peak memory usage of XTandem Parser when reading X!Tandem XML result files of different sizes, reproducing the paper's Table 1 metrics (parsing time vs. number of quality spectra/valid models) and the <1 GB RAM constraint.

## When to use

Use when you need to quantify how quickly and with how much memory a Java parser reads X!Tandem XML output files at different scales (thousands to hundreds of thousands of quality spectra), and to compile the timing/RAM results into a metrics table.

## When NOT to use

- Input results come from other search engines (e.g. MASCOT .dat files or OMSSA result files) — use Mascot-Dat File or OMSSA Parser instead, since XTandem Parser only extracts information from X!Tandem output xml files.
- The goal is visualization or biological analysis of identifications rather than parser performance benchmarking — the paper recommends PeptideShaker as a search engine independent platform for that purpose.
- You need to parse spectra itself rather than an X!Tandem search output; this skill only times reading of already-produced X!Tandem XML result files.

## Inputs

- X!Tandem XML result files of graded sizes (containing 3225, 60902, and 233327 quality spectra)
- XTandem Parser Java library (Apache2 licence; Maven dependency de.proteinms.xtandemparser:xtandem-parser)
- Java runtime with wall-clock timer (e.g. System.nanoTime) and JVM memory tracking or profiler

## Outputs

- performance_metrics.csv with parsing time (s) and peak RAM per file
- Counts of quality spectra and valid models per file (reference: 3225/101, 60902/2065, 233327/7644)

## How to apply

Obtain the XTandem Parser library (Apache2 licence, Maven artifact de.proteinms.xtandemparser:xtandem-parser) and its distributed X!Tandem XML test files. Write a Java script that instantiates the XTandemFile object on each XML result file and reads through all identifications, matching the reported scales of 3225, 60902, and 233327 quality spectra (with 101, 2065, and 7644 valid models). Time each run with a wall-clock timer such as System.nanoTime, and track peak memory via Runtime memory tracking or a JVM profiler to verify the reported constraint that all runs stay below 1 GB of RAM. Compile parsing time (s) and peak RAM per file into performance_metrics.csv. Judge correctness by whether the times approximate the Table 1 references: <1 s for 3225 spectra, 5 s for 60902 spectra, and 10 s for 233327 spectra.

## Related tools

- **XTandem Parser** (Java library being benchmarked; instantiates XTandemFile objects to parse X!Tandem XML result files while timing and memory use are recorded) — github.com/compomics/xtandem-parser
- **Java** (Implementation language for the benchmark script (System.nanoTime timing, Runtime memory tracking))
- **X!Tandem** (Upstream open-source search engine whose XML output files are the benchmark inputs)

## Examples

```
XTandemFile xtandemFile = new XTandemFile("results_3225.xml"); long t0 = System.nanoTime(); /* read all identifications */ long elapsedMs = (System.nanoTime() - t0) / 1000000;
```

## Evaluation signals

- Parsing time for the 3225-spectra file is under 1 second; the 60902-spectra file completes in about 5 seconds; the 233327-spectra file in about 10 seconds.
- Peak RAM stays below 1 GB for every parsing run.
- Number of valid models parsed matches the reference counts: 101, 2065, and 7644 for the three files.
- performance_metrics.csv contains one row per input file with parsing time and peak RAM populated for all three file sizes.

## Limitations

- Reference timings are tied to the specific test dataset sizes distributed with XTandem Parser (3225, 60902, 233327 quality spectra); results on other files are not covered by Table 1.
- The reported value for the smallest file appears as '<1 s' (rendered 'o1' in the extracted table), so exact sub-second timings cannot be compared precisely.
- The original distribution URL (http://xtandem-parser.googlecode.com) is superseded by the GitHub repository; the README's Maven snippet uses placeholder version 'X.Y.Z' that must be updated to the latest release.
- No discussion section content is available in the source document, so caveats beyond the abstract's performance claims cannot be traced.

## Evidence
