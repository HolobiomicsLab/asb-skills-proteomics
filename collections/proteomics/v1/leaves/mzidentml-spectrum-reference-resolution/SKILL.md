---
name: mzidentml-spectrum-reference-resolution
description: Use when an mzIdentML file must be linked to spectra in external peak
  list files (by spectrum index, spectrum ID, or scan number) and a uniform lookup
  across multiple MS file formats is needed in Java.
when_to_use_negative:
- The spectra are embedded in the identification file itself (not referenced externally),
  so no external peak list resolution is needed
- You need de novo conversion between formats rather than spectrum lookup — jmzReader-gui's
  mgf export is the supported route for format conversion, not this interface
- The working language is not Java/JVM and no Maven-based dependency on jmzReader
  is possible
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Java parser library whose common interface resolves the mzIdentML spectrum
    reference methods (by index, ID, scan number) across all supported peak list formats
  repo: http://code.google.com/p/jmzreader/
- name: Maven
  role: Build/dependency system used to include jmzReader parsers via the pom.xml
    dependency and repository snippets
- name: jmzReader-gui
  role: Viewer that demonstrates usage of the jmzReader interface and can export all
    supported file formats to mgf for spot-checking lookups
  repo: http://code.google.com/p/jmzreader/
- name: jmzML
  role: Existing API on which the jmzReader mzML parser is based
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:35767b72875cd0b0dd5707e2ce6055f3e939a0cdb3d5bfe1a1c02dbf8fb4133d
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mzidentml-spectrum-reference-resolution

## Summary

A Java-based skill for resolving mzIdentML external spectrum references against peak list files using the jmzReader library's single common parser interface, so one code path handles DTA, MS2, MGF, PKL, mzXML, mzData, mzML, and PRIDE XML inputs.

## When to use

Use when an mzIdentML file must be linked to spectra in external peak list files (by spectrum index, spectrum ID, or scan number) and a uniform lookup across multiple MS file formats is needed in Java.

## When NOT to use

- The spectra are embedded in the identification file itself (not referenced externally), so no external peak list resolution is needed
- You need de novo conversion between formats rather than spectrum lookup — jmzReader-gui's mgf export is the supported route for format conversion, not this interface
- The working language is not Java/JVM and no Maven-based dependency on jmzReader is possible

## Inputs

- mzIdentML file with spectrum references (SpectrumIdentificationResult / spectraData references)
- External peak list files in DTA, MS2, MGF, PKL, mzXML, mzData, mzML, or PRIDE XML format
- Maven project (pom.xml) configured with the EBI pst-release repository

## Outputs

- Spectrum objects (peak lists with precursor information where available) retrieved via the common jmzReader interface
- Structured resolution report mapping each format/method combination (byIndex, byID, byScanNumber) to the resolved spectrum

## How to apply

Add jmzReader as a Maven dependency (uk.ac.ebi.pride.tools:jmzreader) resolving from the EBI pst-release repository, and instantiate the parser matching the peak list format at hand (DTA, MS2, MGF, PKL, mzXML, mzData, mzML, or PRIDE XML). Because mzIdentML files contain no spectra data but refer to external peak list files, call the common jmzReader interface's three mzIdentML-style reference methods — lookup by spectrum index, by spectrum ID, and by scan number — rather than writing format-specific code. Record, per format and per method, the spectrum returned (e.g. peak count and precursor information where available) into a structured report. Validate that each reference method resolves to the expected spectrum in every supported format, flagging any failing format/method combination. Rationale: all jmzReader parsers are optimized for use with mzIdentML and share one interface, and a custom line-by-line text parsing class lets them handle arbitrarily large files in minimal memory.

## Related tools

- **jmzReader** (Java parser library whose common interface resolves the mzIdentML spectrum reference methods (by index, ID, scan number) across all supported peak list formats) — http://code.google.com/p/jmzreader/
- **Maven** (Build/dependency system used to include jmzReader parsers via the pom.xml dependency and repository snippets)
- **jmzReader-gui** (Viewer that demonstrates usage of the jmzReader interface and can export all supported file formats to mgf for spot-checking lookups) — http://code.google.com/p/jmzreader/
- **jmzML** (Existing API on which the jmzReader mzML parser is based)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>latest</version><type>jar</type></dependency>
```

## Evaluation signals

- Every reference method (byIndex, byID, byScanNumber) resolves to the expected spectrum in every supported format (DTA, MS2, MGF, PKL, mzXML, mzData, mzML)
- Any format/method combination that fails lookup is explicitly flagged in the resolution report rather than silently returning an empty spectrum
- Resolved spectra report sane properties: consistent peak counts and precursor information where the format provides it
- The Maven build resolves uk.ac.ebi.pride.tools:jmzreader from the EBI pst-release repository with the version number updated to the latest version
- Memory footprint stays low on large peak list files, consistent with the line-by-line parsing design

## Limitations

- jmzIdentML files contain no spectra themselves; the skill only resolves references to external files and cannot recover spectra from the mzIdentML document
- Version numbers in the README Maven snippet are placeholders — 'you need to change the version number to the latest version'
- The README contains a malformed pst-release repository URL (double 'h' in https://ww.ebi.ac.uk), so repository configuration may need correction
- The library is still evolving; developers are advised the API may expand, and latest source is in the SVN repository

## Evidence
