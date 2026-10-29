---
name: precursor-ion-information-parsing
description: Use when an mzXML (or other supported peak-list/XML-based) MS file must
  be parsed in Java to recover scan records with precursor ion details and peak arrays
  for downstream proteomics analysis.
when_to_use_negative:
- Input is an mzIdentML identification file — mzIdentML files do not contain spectra
  data but only reference external peak list files, so there is nothing to extract
  scan/precursor/peak data from directly.
- A one-off viewer/inspection task is needed rather than programmatic extraction —
  the jmzReader-gui visualizes peak list files and exports them to mgf format instead.
- Input spectra are already available in a format handled by a different dedicated
  API and re-implementing via the common jmzReader interface adds no benefit.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Java API collection providing the mzXML (and mzData/mzML/DTA/MS2/MGF/PKL)
    parsers with a common interface for iterating scans and extracting precursor and
    peak data
  repo: https://github.com/PRIDE-Utilities/jmzReader
- name: jmzML
  role: Underlying API on which jmzReader's mzML support is based, enabling the same
    scan/precursor extraction on mzML inputs
- name: Java
  role: Programming language in which the jmzReader parsing workflow is implemented
- name: Maven
  role: Build/dependency management used to pull the uk.ac.ebi.pride.tools jmzReader
    artifact into the project
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:72f5bb0146b9730cfe5e332ac590c9d0a2da9c32d2861ad0402c2988f72fdcd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PRIDE-Utilities/jmzReader
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# precursor-ion-information-parsing

## Summary

Method for parsing mzXML files with the jmzReader Java library to extract scan numbers, MS levels, precursor ion information (m/z, charge, intensity), and m/z–intensity peak lists from MS/MS scans, alongside DTA, MS2, MGF, PKL, mzData, and mzML formats.

## When to use

Use when an mzXML (or other supported peak-list/XML-based) MS file must be parsed in Java to recover scan records with precursor ion details and peak arrays for downstream proteomics analysis.

## When NOT to use

- Input is an mzIdentML identification file — mzIdentML files do not contain spectra data but only reference external peak list files, so there is nothing to extract scan/precursor/peak data from directly.
- A one-off viewer/inspection task is needed rather than programmatic extraction — the jmzReader-gui visualizes peak list files and exports them to mgf format instead.
- Input spectra are already available in a format handled by a different dedicated API and re-implementing via the common jmzReader interface adds no benefit.

## Inputs

- mzXML file (MS peak list / XML-based MS data)
- Optionally other jmzReader-supported formats: DTA, MS2, MGF, PKL, mzData, mzML, PRIDE XML
- Java project with Maven build configuration

## Outputs

- Scan records (e.g. JSON array) containing scan number, MS level, precursor ion information (m/z, charge, intensity), and decoded m/z–intensity peak list, preserving scan order

## How to apply

Add jmzReader as a Maven dependency (group uk.ac.ebi.pride.tools, updating the version number to the latest release) in your Java project. Load the mzXML file with the jmzReader mzXML reader, which implements the common jmzReader interface used by all parsers. Iterate over every scan element, extracting the scan index/number and MS level from the scan attributes. For each MS2 (or higher-level) scan, pull the precursor ion information — precursor m/z, charge, and intensity — from the precursor selection element, then decode the m/z–intensity peak array and attach it to the scan record. Because jmzReader's parsers implement a common interface that supports the methods mzIdentML uses to reference external spectra, the same iteration logic transfers to MGF, mzData, and mzML (the latter based on jmzML) inputs. Write all scan records in original scan order and validate against the source file's scan count.

## Related tools

- **jmzReader** (Java API collection providing the mzXML (and mzData/mzML/DTA/MS2/MGF/PKL) parsers with a common interface for iterating scans and extracting precursor and peak data) — https://github.com/PRIDE-Utilities/jmzReader
- **jmzML** (Underlying API on which jmzReader's mzML support is based, enabling the same scan/precursor extraction on mzML inputs)
- **Java** (Programming language in which the jmzReader parsing workflow is implemented)
- **Maven** (Build/dependency management used to pull the uk.ac.ebi.pride.tools jmzReader artifact into the project)

## Examples

```
MzXMLFile mzxml = new MzXMLFile(new File("test.mzXML")); for (Scan scan : mzxml) { int num = scan.getNum(); int msLevel = scan.getMsLevel(); }
```

## Evaluation signals

- The output file exists and contains exactly one record per scan in the source mzXML file.
- Each record carries the required fields: scan number, MS level, precursor ion information, and a decoded m/z–intensity peak list.
- Records appear in the same order as scans in the source mzXML document.
- Precursor fields (m/z, charge, intensity) are populated only where the source scan declares a precursor selection element (MS2 or higher scans).
- Peak arrays re-encode to the same m/z and intensity values as the raw mzXML scan data when compared record-by-record.

## Limitations

- The README's Maven snippet uses a placeholder version — you need to change the version number to the latest version, or the dependency will not resolve.
- The library is still evolving; parsers and APIs may change between releases.
- mzML parsing in jmzReader depends on the separate jmzML API, so behavior for mzML inputs is inherited from that module.
- Parsers are optimized for mzIdentML spectrum referencing; arbitrary custom XML attributes outside the common interface may require direct access to format-specific classes.

## Evidence
