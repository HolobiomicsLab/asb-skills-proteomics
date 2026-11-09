---
name: spectrum-header-field-extraction
description: Use when input is an MS2 (or other peak list) file from a proteomics
  workflow and each spectrum's charge state, precursor m/z, retention time, and peak
  list must be extracted into structured records, e.g. for mzIdentML-referenced spectra.
when_to_use_negative:
- The spectra are already embedded in the identification file (mzIdentML does not
  contain spectra data but refers to external peak list files), so no peak list parsing
  is needed
- The input is an XML-based format you cannot represent with jmzReader's common interface
  and you need format-specific semantics beyond spectrum-level header/peak extraction
- You need interactive visualization or format conversion to mgf — the jmzReader-gui
  tool is the article's designated route for viewing and exporting peak list files
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: jmzReader
  role: Reference Java API library providing the MS2 (and dta, mgf, pkl, mzData, mzXML,
    mzML, PRIDE XML) parsers and the custom line-by-line text parsing class on which
    the extraction implementation is modeled
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: Java
  role: Implementation language for the line-by-line parser that extracts spectrum
    headers and peak lists and serializes them to ms2_spectra.
- name: Maven
  role: Build/dependency mechanism used to add all jmzReader parsers to the project
    via its pom.xml
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:859168031cdf195cb89c3665d8b3c05d278ae73d6c9cc3371ab386135e805f55
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PRIDE-Utilities/jmzReader
  tools:
  - Java
  - Maven
  - jmzReader
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spectrum-header-field-extraction

## Summary

Parses MS2 peak list files (and related formats such as DTA, MGF, PKL, mzXML, mzData, mzML) to emit one structured record per spectrum containing header fields (charge state, precursor m/z, retention time) and the associated m/z–intensity peak list, using the jmzReader Java APIs as the reference implementation.

## When to use

Use when input is an MS2 (or other peak list) file from a proteomics workflow and each spectrum's charge state, precursor m/z, retention time, and peak list must be extracted into structured records, e.g. for mzIdentML-referenced spectra.

## When NOT to use

- The spectra are already embedded in the identification file (mzIdentML does not contain spectra data but refers to external peak list files), so no peak list parsing is needed
- The input is an XML-based format you cannot represent with jmzReader's common interface and you need format-specific semantics beyond spectrum-level header/peak extraction
- You need interactive visualization or format conversion to mgf — the jmzReader-gui tool is the article's designated route for viewing and exporting peak list files

## Inputs

- MS2 peak list file (flat text, with per-spectrum headers and m/z–intensity peak pairs)
- Optionally other jmzReader-supported peak list formats: DTA, MGF, PKL, mzXML, mzData, mzML, PRIDE XML
- Maven pom.xml of the consuming Java project

## Outputs

- ms2_spectra. — one structured JSON record per spectrum containing charge state, precursor m/z, retention time (when present), and the peak list (m/z and intensity pairs)

## How to apply

Obtain a sample MS2 file (e.g. from the jmzReader repository test resources) and add jmzReader as a Maven dependency (groupId uk.ac.ebi.pride.tools, artifactId jmzreader). In Java, implement a line-by-line text parser modeled on the jmzReader custom parsing class, which per the paper can handle arbitrarily large files in minimal memory. Parse the MS2 header structure to extract, for each spectrum, the charge state, precursor m/z, and retention time when the field is present, then parse the associated peak list of m/z and intensity pairs following the conventions of the jmzReader MS2 reader. Emit one structured record per spectrum (header fields plus peak list) serialized as JSON. The rationale for using jmzReader's common interface is that mzIdentML files reference external peak list files in multiple formats, and a single interface covers dta, mgf, ms2, mzData, mzXML, pkl, mzML, and PRIDE XML.

## Related tools

- **jmzReader** (Reference Java API library providing the MS2 (and dta, mgf, pkl, mzData, mzXML, mzML, PRIDE XML) parsers and the custom line-by-line text parsing class on which the extraction implementation is modeled) — https://github.com/PRIDE-Utilities/jmzReader
- **Java** (Implementation language for the line-by-line parser that extracts spectrum headers and peak lists and serializes them to ms2_spectra.)
- **Maven** (Build/dependency mechanism used to add all jmzReader parsers to the project via its pom.xml)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>LATEST</version></dependency>
```

## Evaluation signals

- ms2_spectra. exists and is valid JSON
- The file contains exactly one record per spectrum in the input MS2 file
- Each record carries the charge state, precursor m/z, and retention time fields when those fields are present in the input file
- Each record's peak list contains m/z and intensity pairs matching the peaks in the source spectrum
- The parser handles arbitrarily large files without exhausting memory (jmzReader's custom line-by-line parsing behavior)

## Limitations

- jmzReader version number must be set manually to the latest version when adding the Maven dependency (no fixed version is pinned in the README snippet)
- Retention time is only emitted when the field is present in the MS2 file; missing header fields yield absent values in the records
- The library is still evolving; jmzReader's own README notes 'we are committed to expand this library and add more useful classes'

## Evidence
