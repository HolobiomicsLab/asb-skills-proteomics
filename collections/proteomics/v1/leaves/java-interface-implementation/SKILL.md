---
name: java-interface-implementation
description: Use when building Java software that must read several MS peak list formats
  (e.g. for mzIdentML consumers) and you want one parser interface (lookup by index,
  ID, scan number) instead of per-format code paths.
when_to_use_negative:
- The input is a single fixed file format only — using the abstraction layer adds
  indirection without benefit.
- The client must access format-specific metadata not exposed by the common interface
  (e.g. raw mzData binary block internals).
- Working outside the JVM — the jmzReader parsers are Java APIs and cannot be used
  directly from other runtimes.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: jmzReader
  role: The Java parser library whose parsers all implement the common interface supporting
    mzIdentML's external spectra referencing methods
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: jmzML
  role: Existing API on which the jmzReader mzML parser is based
- name: jmzReader-gui
  role: Demonstrates usage of the jmzReader interface; supports all library formats
    and exports to mgf
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: Maven
  role: Build/dependency mechanism for adding the jmzReader parsers to a Java project
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:9cdc5f1b8f2949d5e400d5cf225d94b9fb49bdc95f00642d9057c7eecc93ffdd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PRIDE-Utilities/jmzReader
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# java-interface-implementation

## Summary

Implement a single common Java interface (as in jmzReader) behind which multiple format-specific parsers expose identical operations, so client code supports one API instead of many file formats. In jmzReader, all peak list parsers (DTA, MS2, MGF, PKL, mzXML, mzData, mzML) share one interface that supports mzIdentML's external spectra referencing methods.

## When to use

Use when building Java software that must read several MS peak list formats (e.g. for mzIdentML consumers) and you want one parser interface (lookup by index, ID, scan number) instead of per-format code paths.

## When NOT to use

- The input is a single fixed file format only — using the abstraction layer adds indirection without benefit.
- The client must access format-specific metadata not exposed by the common interface (e.g. raw mzData binary block internals).
- Working outside the JVM — the jmzReader parsers are Java APIs and cannot be used directly from other runtimes.

## Inputs

- MS peak list files in any supported format: DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML
- mzIdentML files referencing external spectra (by index, ID, or scan number)

## Outputs

- Parsed spectrum objects (spectrum identifier, CV parameters, m/z-intensity peak arrays) returned through the common interface
- Per-format, per-reference-method resolution records (e.g. a resolution report flagging failed lookups)

## How to apply

Set up a Java project and add the parser library as a Maven dependency (uk.ac.ebi.pride.tools:jmzreader) from the EBI pst-release Nexus repository. Define or use the single common interface that abstracts format differences, and implement one parser class per format (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML) behind it. In jmzReader, the interface must support mzIdentML's external spectra referencing methods — lookup by spectrum index, by spectrum ID, and by scan number — because mzIdentML files contain no spectra themselves but refer to external peak list files. For memory efficiency with large files, use a custom line-by-line text file parsing class so arbitrary large files can be handled in minimal memory. Expose shared results (spectrum peaks, precursor info) through the interface so clients never touch format-specific classes.

## Related tools

- **jmzReader** (The Java parser library whose parsers all implement the common interface supporting mzIdentML's external spectra referencing methods) — https://github.com/PRIDE-Utilities/jmzReader
- **jmzML** (Existing API on which the jmzReader mzML parser is based)
- **jmzReader-gui** (Demonstrates usage of the jmzReader interface; supports all library formats and exports to mgf) — https://github.com/PRIDE-Utilities/jmzReader
- **Maven** (Build/dependency mechanism for adding the jmzReader parsers to a Java project)

## Examples

```
<dependency>
  <groupId>uk.ac.ebi.pride.tools</groupId>
  <artifactId>jmzreader</artifactId>
</dependency>
```

## Evaluation signals

- Every reference method (byIndex, byID, byScanNumber) resolves to the expected spectrum in every supported format (DTA, MS2, MGF, PKL, mzXML, mzData, mzML); any failing format/method combination is flagged in the report.
- Each parsed spectrum record carries a spectrum identifier, CV parameter fields, and non-empty m/z-intensity peak arrays.
- Client code compiles against only the common interface, with no imports of format-specific parser classes.
- Arbitrarily large peak list files are parsed within minimal, bounded memory (line-by-line parsing path is exercised).
- Per-format JSON/structured output contains exactly one record per spectrum in the input file.

## Limitations

- The library was still evolving at publication; the README notes 'the library is still evolving' and developers must update the Maven version number to the latest version.
- mzIdentML files do not contain spectra data; resolution depends on locating the external peak list files referenced by the mzIdentML entry.
- Format coverage is limited to the parsers shipped (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML); unsupported formats require new parser implementations behind the interface.

## Evidence
