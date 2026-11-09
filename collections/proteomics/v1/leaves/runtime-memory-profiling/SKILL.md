---
name: runtime-memory-profiling
description: Use when benchmarking X!Tandem XML result-file parsing performance —
  measuring seconds and peak memory for files of ~3225 to ~223327 quality spectra
  against published reference timings.
when_to_use_negative:
- Input files are not X!Tandem XML output (e.g. Mascot-Dat or OMSSA result files)
  — use Mascot-Dat File or OMSSA Parser instead
- The goal is peptide/protein identification result analysis or visualization rather
  than performance measurement — use PeptideShaker or XTandem Viewer
- Runtime is not a JVM (no Java System.nanoTime / Runtime memory tracking available)
  since the parser is a Java library
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: XTandem Parser
  role: 'Java library under benchmark: instantiated as XTandemFile on each X!Tandem
    XML result file to measure parsing time and peak memory'
  repo: http://xtandem-parser.googlecode.com
- name: Java
  role: Runtime environment providing System.nanoTime wall-clock timing and JVM memory
    tracking/profiling for peak-RAM measurement
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.200900759
    title: 'XTandem Parser: An open‐source library to parse and analyse X!Tandem MS/MS
      search results'
  source_content_hash: sha256:361e3b2548407939af0f8e829a41858eef991b61f993116abfac75db394c4425
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - XTandem Parser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# runtime-memory-profiling

## Summary

Benchmark wall-clock parsing time and peak RAM of XTandem Parser runs on X!Tandem XML result files of graded sizes, reproducing the paper's Table 1 performance metrics (parsing time vs. number of quality spectra) and verifying the <1 GB RAM constraint.

## When to use

Use when benchmarking X!Tandem XML result-file parsing performance — measuring seconds and peak memory for files of ~3225 to ~223327 quality spectra against published reference timings.

## When NOT to use

- Input files are not X!Tandem XML output (e.g. Mascot-Dat or OMSSA result files) — use Mascot-Dat File or OMSSA Parser instead
- The goal is peptide/protein identification result analysis or visualization rather than performance measurement — use PeptideShaker or XTandem Viewer
- Runtime is not a JVM (no Java System.nanoTime / Runtime memory tracking available) since the parser is a Java library

## Inputs

- X!Tandem XML result files of graded sizes (approximately 3225, 60902, and 223327 quality spectra)
- XTandem Parser library (Apache2 licence) and its bundled test dataset
- Java runtime environment (JVM)

## Outputs

- performance_metrics.csv with columns: number of quality spectra, number of valid models, parsing time (s), peak RAM
- Per-run wall-clock parsing times (<1 s, 5 s, 10 s reference) and peak-memory readings

## How to apply

Obtain the XTandem Parser Java library (Apache2 licence) and its distributed test dataset from the repository, then write a script that instantiates the XTandemFile object on each X!Tandem XML result file and reads through all identifications, matching the reference scale of 3225, 60902, and 223327 quality spectra (with 101, 2065, and 7644 valid models). Time each parsing run with a wall-clock timer such as System.nanoTime in Java and record the number of valid models parsed. Measure peak memory per run via JVM Runtime memory tracking or a memory profiler to test the reported constraint that all runs stay below 1 GB of RAM. Compile parsing time and peak RAM per file into a metrics table (e.g. performance_metrics.csv) mirroring the paper's Table 1 layout of quality-spectra count, valid-model count, and parsing time in seconds.

## Related tools

- **XTandem Parser** (Java library under benchmark: instantiated as XTandemFile on each X!Tandem XML result file to measure parsing time and peak memory) — http://xtandem-parser.googlecode.com
- **Java** (Runtime environment providing System.nanoTime wall-clock timing and JVM memory tracking/profiling for peak-RAM measurement)

## Examples

```
XTandemFile xtf = new XTandemFile("tandem_results.xml"); long t0 = System.nanoTime(); while (xtf.hasNextIdentification()) { xtf.nextIdentification(); } long elapsed = (System.nanoTime() - t0) / 1000000000;
```

## Evaluation signals

- Parsing times approximate Table 1 reference values: <1 s for 3225 spectra, 5 s for 60902 spectra, 10 s for 223327 spectra
- Peak RAM remains below 1 GB for every run, matching the paper's claim that all tests run in less than 1 GB of RAM
- Number of valid models parsed matches reference counts: 101, 2065, and 7644 for the three file sizes respectively
- performance_metrics.csv has one row per input file with quality-spectra count, valid-model count, parsing time (s), and peak RAM populated

## Limitations

- Reference timings (<1 s, 5 s, 10 s) are hardware- and JVM-dependent; only approximate agreement, not exact reproduction, is expected
- Only three file sizes were benchmarked in the paper, so scaling behavior outside 3225–223327 spectra is not characterized
- The article provides no discussion-section content clarifying edge cases or profiling methodology details

## Evidence
