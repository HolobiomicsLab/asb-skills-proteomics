---
name: mzxml-spectrum-xml-parsing
description: Use when the input is an mzXML peak list file and you need scan-level
  spectra (scan numbers, precursor m/z/charge/intensity, peak lists) in a structured
  form, e.g. to feed spectra referenced by mzIdentML.
when_to_use_negative:
- The input spectra are already in a text peak list format (dta, mgf, ms2, pkl) or
  another XML format (mzData, mzML, PRIDE XML) better handled by a dedicated jmzReader
  parser
- You need raw (profile-mode) instrument data or chromatograms rather than peak lists
  — jmzReader parses peak list and XML-based peak list formats, not raw acquisitions
- You are working outside the JVM ecosystem and cannot use Java/Maven tooling
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Java API whose mzXML parser implements the common jmzReader interface to extract
    scans, precursor info, and peak lists from mzXML files
  repo: http://code.google.com/p/jmzreader/
- name: jmzML
  role: Underlying API on which jmzReader's mzML parser is based; relevant sibling
    format if input is mzML instead of mzXML
- name: Maven
  role: Build/dependency system used to add the jmzReader parsers to a project via
    EBI Nexus repository entries
- name: jmzReader-gui
  role: 'Visualization/verification tool: views mzXML peak list files and demonstrates
    usage of the jmzReader interface, and can export to mgf'
  repo: http://code.google.com/p/jmzreader/
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:dbdec9c6605c8f2811ef3709bc154f4e6e9646e3a9e553406c7d53e07c6e15b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Maven
  - jmzML
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mzxml-spectrum-xml-parsing

## Summary

Parses mzXML XML-based MS peak list files using the jmzReader Java API, extracting scan numbers, MS levels, precursor ion information, and decoded m/z-intensity peak arrays. Part of jmzReader's collection of parsers for DTA, MS2, MGF, PKL, mzXML, mzData, and mzML formats.

## When to use

Use when the input is an mzXML peak list file and you need scan-level spectra (scan numbers, precursor m/z/charge/intensity, peak lists) in a structured form, e.g. to feed spectra referenced by mzIdentML.

## When NOT to use

- The input spectra are already in a text peak list format (dta, mgf, ms2, pkl) or another XML format (mzData, mzML, PRIDE XML) better handled by a dedicated jmzReader parser
- You need raw (profile-mode) instrument data or chromatograms rather than peak lists — jmzReader parses peak list and XML-based peak list formats, not raw acquisitions
- You are working outside the JVM ecosystem and cannot use Java/Maven tooling

## Inputs

- mzXML file (XML-based MS peak list format)
- Java/Maven project with uk.ac.ebi.pride.tools:jmzreader dependency from EBI Nexus

## Outputs

- Per-scan records with scan number, MS level, precursor ion info (m/z, charge, intensity), and decoded m/z-intensity peak list (e.g. a JSON array of scan records in scan order)

## How to apply

Set up a Java/Maven project and add the jmzReader dependency (group uk.ac.ebi.pride.tools, artifact jmzreader) using the EBI Nexus repositories, substituting the latest version number. Instantiate the jmzReader mzXML reader — it implements the common jmzReader interface shared by all parsers (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML) — and load the mzXML file. Iterate over every scan element, reading the scan index/number and MS level from the scan attributes; for MS2 (or higher) scans, extract precursor ion m/z, charge, and intensity from the precursor selection element. Decode each scan's binary m/z-intensity peak array and attach it to the scan record, preserving scan order. Because all parsers share one interface and support mzIdentML's external spectra referencing methods, the resulting records can be exposed to mzIdentML-based software without format-specific code.

## Related tools

- **jmzReader** (Java API whose mzXML parser implements the common jmzReader interface to extract scans, precursor info, and peak lists from mzXML files) — http://code.google.com/p/jmzreader/
- **jmzML** (Underlying API on which jmzReader's mzML parser is based; relevant sibling format if input is mzML instead of mzXML)
- **Maven** (Build/dependency system used to add the jmzReader parsers to a project via EBI Nexus repository entries)
- **jmzReader-gui** (Visualization/verification tool: views mzXML peak list files and demonstrates usage of the jmzReader interface, and can export to mgf) — http://code.google.com/p/jmzreader/

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>1.2.3</version></dependency> // then: MzXMLFile reader = new MzXMLFile(new File("sample.mzXML"))
```

## Evaluation signals

- The output contains exactly one record per scan in the source mzXML, in original scan order
- Every MS2+ record carries precursor information (precursor m/z, charge, intensity) extracted from the precursor selection element
- Each record includes a non-empty decoded m/z-intensity peak array consistent with the mzXML binary data block
- Records expose the scan number and MS level fields read from the scan attributes
- The parsed spectra are retrievable through the common jmzReader interface, i.e. equivalent to loading the same file via jmzReader-gui or another jmzReader parser

## Limitations

- jmzReader version must be manually set to the latest version in the Maven dependency (README notes 'you need to change the version number to the latest version')
- mzIdentML files do not contain spectra data themselves — spectra must come from the external peak list file via the parser's interface
- Parsers handle arbitrarily large files in minimal memory only via the custom line-by-line text parsing class; memory behavior depends on this implementation
- The library is still evolving; the README states they are committed to expand the library and add more classes

## Evidence
