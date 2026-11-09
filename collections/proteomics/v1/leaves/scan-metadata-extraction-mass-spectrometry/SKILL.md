---
name: scan-metadata-extraction-mass-spectrometry
description: Use when input is an mzXML (or other jmzReader-supported XML-based MS)
  file and the task requires per-scan records with scan numbers, MS levels, precursor
  m/z/charge/intensity, and decoded peak arrays.
when_to_use_negative:
- Input is an mzIdentML file — mzIdentML files do not contain spectra data but refer
  to external peak list files, so scan metadata must be fetched from the referenced
  peak list file instead
- Input is raw instrument data in a vendor binary format not among the supported peak
  list/XML formats (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML)
- The task is format conversion to mgf only — the jmzReader-gui tool already exports
  all supported formats into mgf without custom parsing
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Java API used to parse the mzXML file and expose scan numbers, MS levels,
    precursor information, and peak lists via a common parser interface
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Programming language in which the jmzReader parsing code is implemented and
    invoked
- name: Maven
  role: Build system used to add the jmzReader parsers as project dependencies from
    the EBI Nexus repository
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: jmzML
  role: Underlying API on which jmzReader's mzML parser is based, relevant when the
    input is mzML rather than mzXML
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:8cd1440f83c1400367552f7c1f0d00cf38b9820007d01325c3b68c449e44008d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scan-metadata-extraction-mass-spectrometry

## Summary

Extracts scan-level metadata (scan numbers, MS levels, precursor ion information) and m/z-intensity peak lists from XML-based MS data files such as mzXML, using the jmzReader Java parser library.

## When to use

Use when input is an mzXML (or other jmzReader-supported XML-based MS) file and the task requires per-scan records with scan numbers, MS levels, precursor m/z/charge/intensity, and decoded peak arrays.

## When NOT to use

- Input is an mzIdentML file — mzIdentML files do not contain spectra data but refer to external peak list files, so scan metadata must be fetched from the referenced peak list file instead
- Input is raw instrument data in a vendor binary format not among the supported peak list/XML formats (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML)
- The task is format conversion to mgf only — the jmzReader-gui tool already exports all supported formats into mgf without custom parsing

## Inputs

- mzXML file (XML-based MS peak list format)
- Other jmzReader-supported formats: DTA, MS2, MGF, PKL, mzData, mzML, PRIDE XML

## Outputs

- Per-scan records containing scan number, MS level, precursor ion m/z, charge, and intensity
- Decoded m/z-intensity peak lists per scan (e.g. written as a JSON array in scan order)

## How to apply

Set up a Java parsing environment by adding jmzReader as a Maven dependency (groupId uk.ac.ebi.pride.tools, artifactId jmzreader, repository EBI Nexus pst-release). Load the mzXML file with the jmzReader mzXML reader, which implements the common jmzReader interface shared by all format parsers (DTA, MS2, MGF, PKL, mzData, mzXML, mzML, PRIDE XML). Iterate over every scan element and read the scan index/number and MS level from the scan attributes; for each MS2-or-higher scan, extract precursor ion m/z, charge, and intensity from the precursor selection element. Decode the m/z-intensity peak array for each scan and attach it to the scan record, preserving scan order. jmzReader's parsers are built on a custom line-by-line parsing class, so arbitrarily large files can be handled in minimal memory. Validate that the output contains one record per scan, each carrying scan number, MS level, precursor info, and peak list.

## Related tools

- **jmzReader** (Java API used to parse the mzXML file and expose scan numbers, MS levels, precursor information, and peak lists via a common parser interface) — http://code.google.com/p/jmzreader/
- **Java** (Programming language in which the jmzReader parsing code is implemented and invoked)
- **Maven** (Build system used to add the jmzReader parsers as project dependencies from the EBI Nexus repository) — https://github.com/PRIDE-Utilities/jmzReader
- **jmzML** (Underlying API on which jmzReader's mzML parser is based, relevant when the input is mzML rather than mzXML)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>version</version><type>jar</type></dependency>
```

## Evaluation signals

- The output file exists and contains exactly one record per scan element in the source mzXML
- Each record carries the fields scan number, MS level, precursor info (m/z, charge, intensity), and peak list
- Records are preserved in source scan order when written (e.g. as a JSON array to mzxml_scan_records)
- Precursor information is present only for MS2 (or higher-level) scans, not for MS1 survey scans
- Sample mzXML test resources from the jmzReader repository can be loaded successfully through the common jmzReader interface

## Limitations

- The README notes the library is still evolving and does not state a specific version number in the dependency snippet — you must change the version number to the latest version
- mzIdentML files do not contain spectra data but refer to external peak list files, so this skill alone cannot extract scans from an mzIdentML input
- Parsers handle arbitrarily large files in minimal memory only through the custom line-by-line text parsing class; memory behavior for XML formats depends on the reader implementation

## Evidence
