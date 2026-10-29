---
name: ms2-peak-list-parsing
description: 'Use when input is an MS2 (or related peak list: dta, mgf, pkl, mzXML,
  mzData, mzML, PRIDE XML) file and you need one structured record per spectrum with
  header fields and a peak list, especially as spectra for mzIdentML-based analysis.'
when_to_use_negative:
- mzIdentML files themselves — they contain no spectra data but only reference external
  peak list files, so they cannot be parsed for peaks by this skill
- Raw instrument data not in any of jmzReader's supported peak list / XML-based MS
  formats (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML)
- Only converting to a common format rather than structuring records — jmzReader-gui
  already exports all supported formats to mgf directly
edam_operation: http://edamontology.org/operation_0271
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: 'Core parser library: its MS2 reader defines the MS2 parsing conventions,
    and its common interface exposes lookup by spectrum index, ID, and scan number'
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language for the line-by-line MS2 parser and consumers of the
    jmzReader APIs
- name: Maven
  role: 'Dependency management: jmzReader is added via Maven dependency snippets against
    the EBI pst-release repository'
- name: jmzReader-gui
  role: 'Visualization and validation aid: demonstrates usage of the jmzReader interface
    and exports supported formats to mgf'
  repo: https://github.com/PRIDE-Utilities/jmzReader
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:806ba7bc13871a3fbef2ebd462e369315a9abf3e1cbf5613a8d76b54258d5c39
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_0271
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# ms2-peak-list-parsing

## Summary

Parse MS2 peak list files into structured spectrum records (charge state, precursor m/z, retention time, and peak m/z–intensity lists) using jmzReader's line-by-line text parsing approach in Java, handling arbitrarily large files in minimal memory.

## When to use

Use when input is an MS2 (or related peak list: dta, mgf, pkl, mzXML, mzData, mzML, PRIDE XML) file and you need one structured record per spectrum with header fields and a peak list, especially as spectra for mzIdentML-based analysis.

## When NOT to use

- mzIdentML files themselves — they contain no spectra data but only reference external peak list files, so they cannot be parsed for peaks by this skill
- Raw instrument data not in any of jmzReader's supported peak list / XML-based MS formats (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML)
- Only converting to a common format rather than structuring records — jmzReader-gui already exports all supported formats to mgf directly

## Inputs

- MS2 peak list file (sample files available in the jmzReader repository test resources)
- Other supported peak list / XML-based MS formats: DTA, MGF, PKL, mzXML, mzData, mzML, PRIDE XML

## Outputs

- One structured record per spectrum containing header fields (charge state, precursor m/z, retention time when present) and the peak list of m/z and intensity pairs, e.g. serialized to ms2_spectra JSON
- Optional per-format, per-method resolution report recording spectrum returned for each lookup method (byIndex, byID, byScanNumber)

## How to apply

Add jmzReader as a Maven dependency (groupId uk.ac.ebi.pride.tools, artifactId jmzreader, from the EBI pst-release Nexus repository), setting the version to the latest. Obtain a sample MS2 file from the jmzReader repository test resources as parser input. Model the parser on jmzReader's custom-built class that parses text files line by line, which is what allows all parsers to handle arbitrarily large files in minimal memory. For each spectrum, extract the header fields when present in the file — charge state, precursor m/z, and retention time — then parse the associated peak list of m/z and intensity pairs following the MS2 conventions used by the jmzReader MS2 reader. Access spectra through jmzReader's common parser interface, which supports the three mzIdentML-style external-spectra reference methods: lookup by spectrum index, by spectrum ID, and by scan number, so you do not need separate code paths per peak list format.

## Related tools

- **jmzReader** (Core parser library: its MS2 reader defines the MS2 parsing conventions, and its common interface exposes lookup by spectrum index, ID, and scan number) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language for the line-by-line MS2 parser and consumers of the jmzReader APIs)
- **Maven** (Dependency management: jmzReader is added via Maven dependency snippets against the EBI pst-release repository)
- **jmzReader-gui** (Visualization and validation aid: demonstrates usage of the jmzReader interface and exports supported formats to mgf) — https://github.com/PRIDE-Utilities/jmzReader

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>latest</version><type>jar</type></dependency>
```

## Evaluation signals

- ms2_spectra output file exists and is valid JSON
- Output contains exactly one record per spectrum in the input MS2 file
- Every record carries charge state, precursor m/z, and the peak list fields
- Retention time is populated only for spectra where the field is present in the source MS2 file
- Through the common interface, each reference method (byIndex, byID, byScanNumber) resolves to the expected spectrum; any failing format/method combination is flagged in the resolution report

## Limitations

- MS2 header fields such as retention time are only extracted when the field is present in the file; some spectra may lack charge, precursor m/z, or RT
- jmzReader version must be manually set to the latest version in the Maven dependency — the README ships a 'version' placeholder
- The library is still evolving; the README notes the team is committed to expanding it and adding more classes, so API details may change

## Evidence

- [abstract] jmzReader parses MS2 among common formats: [evidence span withheld — anchor status: not_found]
