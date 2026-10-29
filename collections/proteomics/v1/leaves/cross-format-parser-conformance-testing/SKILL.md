---
name: cross-format-parser-conformance-testing
description: Use when validating that peak list parsers across several MS file formats
  (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML) return consistent spectra through
  one API, e.g. for mzIdentML external-spectra referencing.
when_to_use_negative:
- The data are raw instrument output needing conversion to mzML rather than peak list
  parsing for mzIdentML integration
- You only work with one known format and do not need to verify cross-format conformance
  through a common interface
- Spectra access must go through a format-specific API (e.g. jmzML directly) rather
  than the common peak list interface
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Supplies the parsers for each peak list/XML format and the common interface
    through which the three mzIdentML reference methods are invoked
  repo: http://code.google.com/p/jmzreader/
- name: jmzML
  role: Underlying API used by jmzReader for the mzML XML-based MS data format
- name: Maven
  role: Build/dependency mechanism used to pull jmzReader parsers from the EBI Nexus
    repository into the test project
- name: jmzReader-gui
  role: Optional visual cross-check that a parsed file's spectra match what the common
    interface returns
  repo: http://code.google.com/p/jmzreader/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:5fc874c84d2df2504275bbb96b24a5385395ea66a94781a7ab1e05ee0e1be32d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Maven
  - jmzML
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-format-parser-conformance-testing

## Summary

Verify that a multi-format MS parser library (e.g. jmzReader's parsers for DTA, MS2, MGF, PKL, mzXML, mzData, mzML) resolves spectra identically through one common interface under every mzIdentML-style external-spectra reference method (by index, by ID, by scan number), so downstream mzIdentML software needs only a single interface.

## When to use

Use when validating that peak list parsers across several MS file formats (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, PRIDE XML) return consistent spectra through one API, e.g. for mzIdentML external-spectra referencing.

## When NOT to use

- The data are raw instrument output needing conversion to mzML rather than peak list parsing for mzIdentML integration
- You only work with one known format and do not need to verify cross-format conformance through a common interface
- Spectra access must go through a format-specific API (e.g. jmzML directly) rather than the common peak list interface

## Inputs

- Sample peak list files in each supported format: DTA, MS2, MGF, PKL, mzXML, mzData, mzML (and PRIDE XML per the README)
- jmzReader Maven dependency (uk.ac.ebi.pride.tools:jmzreader) from the EBI pst-release repository
- Expected spectra (peak count, precursor info) for each test file

## Outputs

- Per-format, per-method resolution report (e.g. resolution_report) recording which spectrum each reference method returned
- Flagged list of format/method combinations that failed lookup

## How to apply

Set up a Java project and add jmzReader as a Maven dependency from the EBI pst-release Nexus repository (set the version number to the latest release). Load sample peak list files in each supported format (DTA, MS2, MGF, PKL, mzXML, mzData, mzML) from the repository's test resources using the corresponding jmzReader parser. Through the single common jmzReader interface, invoke the three mzIdentML-style spectrum reference lookups — by spectrum index, by spectrum ID, and by scan number — for every file. For each format/method combination, record the spectrum returned (peak count and precursor information where available) in a structured record, then write a per-format, per-method resolution report. Rationale: jmzReader parsers handle arbitrarily large text files in minimal memory via a custom line-by-line parsing class, so lookups should behave uniformly regardless of format size.

## Related tools

- **jmzReader** (Supplies the parsers for each peak list/XML format and the common interface through which the three mzIdentML reference methods are invoked) — http://code.google.com/p/jmzreader/
- **jmzML** (Underlying API used by jmzReader for the mzML XML-based MS data format)
- **Maven** (Build/dependency mechanism used to pull jmzReader parsers from the EBI Nexus repository into the test project)
- **jmzReader-gui** (Optional visual cross-check that a parsed file's spectra match what the common interface returns) — http://code.google.com/p/jmzreader/

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>LATEST</version></dependency>
```

## Evaluation signals

- Every reference method (byIndex, byID, byScanNumber) resolves to the expected spectrum in every supported format
- Any format/method combination that fails lookup is flagged in the resolution report rather than silently passing
- For each file/method pair, the returned spectrum's recorded peak count and precursor information match the known contents of the test file
- The report covers all 7+ formats × 3 reference methods with no missing cells
- Maven build resolves the jmzreader artifact from the pst-release repository with the correct latest version

## Limitations

- jmzReader version number must be set manually to the latest version; the README's dependency snippet ships a placeholder
- The library is still evolving — the README warns the team 'is committed to expand this library and add more useful classes', so parser behavior may change between releases
- Precursor information is only available 'where available' in some peak list formats, so not every format/method cell yields full metadata

## Evidence

- [abstract] all parsers implement a common interface that supports the various methods used by mzIdentML to reference external spectra: "all parsers implement a common interface that supports the various methods used by mzIdentML to reference external spectra"
- [abstract] We here present the jmzReader library: a collection of Java application programming interfaces (APIs) to parse the most commonly used peak list and XML-based mass spectrometry (MS) data formats: "We here present the jmzReader library: a collection of Java application programming interfaces (APIs) to parse the most commonly used peak list and XML-based mass spectrometry (MS) data formats"
- [intro] Based on a custom build class to efficiently parse text files line by line all parsers can handle arbitrary large files in minimal memory: "Based on a custom build class to efficiently parse text files line by line all parsers can handle arbitrary large files in minimal memory"
- [readme] It supports all file formats supported by the jmzReader library and demonstrates the usage of the jmzReader interface.: "It supports all file formats supported by the jmzReader library and demonstrates the usage of the jmzReader interface."
- [readme] Note: you need to change the version number to the latest version.: "Note: you need to change the version number to the latest version."
