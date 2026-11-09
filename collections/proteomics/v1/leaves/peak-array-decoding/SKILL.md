---
name: peak-array-decoding
description: Use when mzIdentML-referenced peak list files (mzData, mzXML, mzML, MGF,
  DTA, MS2, PKL) must be parsed in Java to decode per-spectrum m/z-intensity peak
  arrays, scan/precursor metadata, and cvParam entries.
when_to_use_negative:
- The input is an mzIdentML file and spectra are expected from it — mzIdentML files
  do not contain spectra data but only refer to external peak list files, so decode
  peaks from the referenced file instead.
- A flat MGF file is all the downstream tool needs — jmzReader-gui can export all
  supported file formats directly into the commonly used mgf format, so no custom
  peak-array decoding code is required.
- The goal is archiving or submitting data to PRIDE / ProteomeXchange rather than
  extracting peaks for mzIdentML-based processing — this library parses peak lists,
  it does not handle archive submission.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: jmzReader
  role: Java API whose format parsers (mzData, mzXML, mzML, DTA, MS2, MGF, PKL, PRIDE
    XML) expose spectra and their m/z-intensity peak arrays via a common interface
    for mzIdentML-referenced peak list files
  repo: http://code.google.com/p/jmzreader/
- name: Maven
  role: Build tool used to add the jmzReader parsers (uk.ac.ebi.pride.tools:jmzreader)
    to the project from the EBI Nexus repository
- name: Java
  role: Programming language in which the decoding code runs, using jmzReader's Java
    APIs to iterate spectra and read peak arrays
- name: jmzReader-gui
  role: Viewer to cross-check parsed peak lists and export any supported file format
    to mgf as an independent comparison of the decoded arrays
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:6ffa8faa66f604d63d472fdcc9c93ff80e25c79cbadad95dfb953e84ac553a5b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# peak-array-decoding

## Summary

Decodes per-spectrum m/z-intensity peak arrays and their metadata (spectrum id/index, cvParams, scan number, MS level, precursor ion info) from XML-based MS peak list files such as mzData and mzXML using the jmzReader Java API, so spectra referenced by mzIdentML become reachable through one common interface.

## When to use

Use when mzIdentML-referenced peak list files (mzData, mzXML, mzML, MGF, DTA, MS2, PKL) must be parsed in Java to decode per-spectrum m/z-intensity peak arrays, scan/precursor metadata, and cvParam entries.

## When NOT to use

- The input is an mzIdentML file and spectra are expected from it — mzIdentML files do not contain spectra data but only refer to external peak list files, so decode peaks from the referenced file instead.
- A flat MGF file is all the downstream tool needs — jmzReader-gui can export all supported file formats directly into the commonly used mgf format, so no custom peak-array decoding code is required.
- The goal is archiving or submitting data to PRIDE / ProteomeXchange rather than extracting peaks for mzIdentML-based processing — this library parses peak lists, it does not handle archive submission.

## Inputs

- mzData XML file containing binary peak data blocks (e.g. jmzReader repository test resources)
- mzXML file with scan and precursor selection elements (e.g. jmzReader repository test resources)
- Other jmzReader-supported peak list formats: mzML, DTA, MS2, MGF, PKL, PRIDE XML
- mzIdentML file used to locate its referenced external peak list file

## Outputs

- mzdata_spectra.: one structured JSON record per mzData spectrum with spectrum identifier, CV parameter fields, and non-empty m/z-intensity peak arrays
- mzxml_scan_records.: JSON array of one record per mzXML scan (scan number, MS level, precursor m/z/charge/intensity, peak list) preserving source scan order

## How to apply

Add the jmzReader parsers to a Maven project as uk.ac.ebi.pride.tools:jmzreader (replacing the placeholder version with the latest) resolved through the EBI Nexus repository. Load the file with the format-matching jmzReader reader — the mzData reader for mzData XML test resources or the mzXML reader for mzXML — which implements the common jmzReader interface, so one code path serves all supported formats. Iterate over every spectrum/scan, extracting the spectrum id or index (or the scan number and MS level from mzXML scan attributes) plus the controlled-vocabulary cvParam parameters; for MS2+ mzXML scans also capture precursor m/z, charge, and intensity from the precursor selection element. Decode the m/z and intensity peak arrays stored in the mzData/mzXML binary data blocks and attach them to each record, relying on jmzReader's custom line-by-line parsing to handle arbitrarily large files in minimal memory. Serialize one structured JSON record per spectrum (identifier, CV parameters, peak list) to mzdata_spectra., or per scan (scan number, MS level, precursor info, peak list) to mzxml_scan_records., preserving scan order. Validate that the output is valid JSON, contains exactly one record per spectrum in the source file, and that every record carries a spectrum identifier, CV parameter fields, and non-empty m/z-intensity peak arrays.

## Related tools

- **jmzReader** (Java API whose format parsers (mzData, mzXML, mzML, DTA, MS2, MGF, PKL, PRIDE XML) expose spectra and their m/z-intensity peak arrays via a common interface for mzIdentML-referenced peak list files) — http://code.google.com/p/jmzreader/
- **Maven** (Build tool used to add the jmzReader parsers (uk.ac.ebi.pride.tools:jmzreader) to the project from the EBI Nexus repository)
- **Java** (Programming language in which the decoding code runs, using jmzReader's Java APIs to iterate spectra and read peak arrays)
- **jmzReader-gui** (Viewer to cross-check parsed peak lists and export any supported file format to mgf as an independent comparison of the decoded arrays)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><type>jar</type></dependency>
```

## Evaluation signals

- The output file exists and parses as valid JSON, containing exactly one record per spectrum in the source mzData/mzXML file.
- Every record includes a spectrum identifier (spectrum id/index or scan number) and CV parameter fields.
- Every record carries a non-empty m/z-intensity peak array with matching m/z and intensity lengths, decoded from the file's binary data blocks.
- mzXML records preserve the source scan order, and MS2+ records include precursor m/z, charge, and intensity from the precursor selection element.
- Decoded peak lists agree with jmzReader-gui's MGF export of the same file (comparable peak counts and values).

## Limitations

- mzIdentML files themselves never contain spectra; parsing them yields only references to external peak list files, never peaks.
- The Maven dependency version in the README is a placeholder ('version') that must be changed to the latest jmzReader version or the dependency will not resolve.
- Repository setup requires the EBI Nexus repository, and the README's release-repository URL contains typos, so verify the URL before adding it to the pom.

## Evidence
