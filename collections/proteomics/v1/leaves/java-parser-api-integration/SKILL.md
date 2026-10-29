---
name: java-parser-api-integration
description: Use when a Java workflow must read MS peak list or XML-based MS data
  (mzXML, mzML, mzData, MGF, DTA, MS2, PKL, PRIDE XML) and spectra referenced by mzIdentML
  need uniform programmatic access.
when_to_use_negative:
- Input is raw instrument data (e.g. Thermo RAW) not among the supported text/XML
  formats — convert to mzML or mzXML first
- The task needs spectra embedded in mzIdentML itself — mzIdentML files do not contain
  spectra data but refer to external peak list files
- A non-Java stack is mandated and JNI/cross-language wrapping of the Maven-based
  library is out of scope
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Java API collection providing the mzXML (and other peak list format) parsers
    implementing the common jmzReader interface
  repo: http://code.google.com/p/jmzreader/
- name: jmzML
  role: Underlying API on which jmzReader's mzML parser is based
- name: jmzReader-gui
  role: Viewer demonstrating the jmzReader interface and exporting supported formats
    to mgf
- name: Maven
  role: Build/dependency mechanism for including jmzReader parsers via EBI Nexus repositories
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:3da2f3901995705cd3a12f07ee6d6d3edbb86d75cd210fe93890ddcdf9f124fb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# java-parser-api-integration

## Summary

Integrate the jmzReader Java API collection (parsers for dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML) into a Maven project to parse mass spectrometry peak list files through one common interface, e.g. iterating mzXML scans to extract scan numbers, precursor info, and peak arrays.

## When to use

Use when a Java workflow must read MS peak list or XML-based MS data (mzXML, mzML, mzData, MGF, DTA, MS2, PKL, PRIDE XML) and spectra referenced by mzIdentML need uniform programmatic access.

## When NOT to use

- Input is raw instrument data (e.g. Thermo RAW) not among the supported text/XML formats — convert to mzML or mzXML first
- The task needs spectra embedded in mzIdentML itself — mzIdentML files do not contain spectra data but refer to external peak list files
- A non-Java stack is mandated and JNI/cross-language wrapping of the Maven-based library is out of scope

## Inputs

- mzXML file
- Other supported peak list / XML MS formats: dta, mgf, ms2, mzData, pkl, mzML, PRIDE XML
- Maven pom.xml project

## Outputs

- Scan records with scan number, MS level, precursor info (m/z, charge, intensity), and decoded m/z-intensity peak list (e.g. written as a JSON array preserving scan order)

## How to apply

Add jmzReader as a Maven dependency (group uk.ac.ebi.pride.tools, artifact jmzreader, updating the version number to the latest) and include the EBI Nexus pst-release and pst-snapshots repositories in the pom. Load the input file (e.g. an mzXML file) with the format-specific reader, which implements the common jmzReader interface shared by all parsers so downstream code supports only one interface. Iterate over every scan element, extracting the scan index/number and MS level from scan attributes; for MS2 (or higher-level) scans, pull precursor ion m/z, charge, and intensity from the precursor selection element, then decode the m/z-intensity peak array and attach it to the scan record. All parsers are optimized for use with mzIdentML and reference external spectra via the same interface, and a custom line-by-line text parsing class lets them handle arbitrarily large files in minimal memory.

## Related tools

- **jmzReader** (Java API collection providing the mzXML (and other peak list format) parsers implementing the common jmzReader interface) — http://code.google.com/p/jmzreader/
- **jmzML** (Underlying API on which jmzReader's mzML parser is based)
- **jmzReader-gui** (Viewer demonstrating the jmzReader interface and exporting supported formats to mgf)
- **Maven** (Build/dependency mechanism for including jmzReader parsers via EBI Nexus repositories)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>1.2.3</version></dependency> — then use the mzXML reader implementing the common jmzReader interface
```

## Evaluation signals

- Output contains one record per scan in the source mzXML, preserving scan order
- Each record carries scan number, MS level, precursor info, and peak list fields
- Pom resolves the uk.ac.ebi.pride.tools:jmzreader artifact from the EBI Nexus pst-release/pst-snapshots repositories with the latest version number
- Arbitrarily large peak list files parse within minimal memory (custom line-by-line parsing class)
- Spectra referenced by mzIdentML resolve through the single common parser interface

## Limitations

- Version number in the Maven snippet is a placeholder — must be changed to the latest version
- Library is still evolving; parsers/APIs may expand over time
- mzIdentML files themselves contain no spectra data; parsers only supply externally referenced peak lists

## Evidence

- [readme] <groupId>uk.ac.ebi.pride.tools</groupId>
        <artifactId>jmzreader</artifactId>
        <version>version</version>: "<groupId>uk.ac.ebi.pride.tools</groupId>
        <artifactId>jmzreader</artifactId>
        <version>version</version>"
