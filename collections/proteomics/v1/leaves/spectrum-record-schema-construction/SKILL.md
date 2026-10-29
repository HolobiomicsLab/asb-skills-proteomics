---
name: spectrum-record-schema-construction
description: 'Use when input is a PKL (or similar plain-text peak list: DTA, MS2,
  MGF) mass spectrometry file and spectra must be converted into structured records
  with precursor metadata plus peak lists.'
when_to_use_negative:
- Input is already structured binary/XML MS data such as mzML, mzXML, mzData, or PRIDE
  XML — use the corresponding jmzReader XML parser rather than the PKL line-by-line
  text parser.
- The goal is only to visualize peak list files — use the jmzReader-gui tool instead
  of writing a record schema.
- Spectra must be referenced from mzIdentML via its external spectra referencing methods
  — jmzReader's parsers already provide that interface directly.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Java API providing the PKL peak list parser and the common spectrum record
    interface that the record schema mirrors
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language for the line-by-line PKL parser and spectrum record
    assembly
- name: Maven
  role: Build/dependency mechanism to include the jmzReader parsers via the uk.ac.ebi.pride.tools:jmzreader
    artifact
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:f96df66b5bf6e41a12a4c239df713899e8ddea39d8cf40b835826a25f6fd6fe8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzReader
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectrum-record-schema-construction

## Summary

Constructs structured spectrum records from PKL peak list files by parsing the header line for precursor m/z, charge state, and precursor intensity, then assembling m/z-intensity peak pairs into a uniform record schema, mirroring the common-parser-interface design of the jmzReader library.

## When to use

Use when input is a PKL (or similar plain-text peak list: DTA, MS2, MGF) mass spectrometry file and spectra must be converted into structured records with precursor metadata plus peak lists.

## When NOT to use

- Input is already structured binary/XML MS data such as mzML, mzXML, mzData, or PRIDE XML — use the corresponding jmzReader XML parser rather than the PKL line-by-line text parser.
- The goal is only to visualize peak list files — use the jmzReader-gui tool instead of writing a record schema.
- Spectra must be referenced from mzIdentML via its external spectra referencing methods — jmzReader's parsers already provide that interface directly.

## Inputs

- PKL peak list text file (header line with precursor m/z, charge, intensity followed by m/z-intensity peak lines)
- Optionally other supported peak list formats: DTA, MS2, MGF

## Outputs

- One structured spectrum record per input spectrum containing precursor m/z, charge state, precursor intensity, and a list of m/z-intensity peak pairs (written e.g. to pkl_spectra)

## How to apply

Obtain a sample PKL file from the jmzReader repository test resources (PRIDE-Utilities/jmzReader). Parse each spectrum's header line to extract the precursor m/z, charge state, and precursor intensity fields; PKL headers encode these three values on the line preceding each peak block. Read the subsequent peak lines with a custom line-by-line text file parser — jmzReader's parsers are 'based on a custom build class to efficiently parse text files line by line', which lets arbitrary large files be handled in minimal memory — converting each line into an m/z-intensity pair. Assemble one record per spectrum that combines precursor metadata with the non-empty peak list, mirroring jmzReader's design where 'all parsers implement a common interface that supports the various methods used by mzIdentML to reference external spectra' so downstream code only supports one record schema. Add jmzReader as a Maven dependency (uk.ac.ebi.pride.tools:jmzreader, per the README's EBI Nexus repository configuration) if using the library directly rather than a reimplementation.

## Related tools

- **jmzReader** (Java API providing the PKL peak list parser and the common spectrum record interface that the record schema mirrors) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language for the line-by-line PKL parser and spectrum record assembly)
- **Maven** (Build/dependency mechanism to include the jmzReader parsers via the uk.ac.ebi.pride.tools:jmzreader artifact)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>VERSION</version><type>jar</type></dependency>  // add to pom.xml per README, then instantiate the jmzReader PKL parser on the sample .pkl test resource
```

## Evaluation signals

- Parser runs on the sample PKL file from the jmzReader repository test resources without errors.
- Every emitted spectrum record contains precursor m/z and charge parsed from the PKL header line.
- Every spectrum record has a non-empty peak list consistent with the number of peak lines in the source file.
- Parsed peak m/z-intensity pairs match the numeric values in the source PKL peak lines.
- All records conform to one uniform schema, consistent with jmzReader's common-parser-interface design.

## Limitations

- jmzReader covers only the listed formats (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML); other peak list formats are out of scope.
- The Maven dependency snippet in the README leaves the version number as a placeholder — 'you need to change the version number to the latest version'.
- The library is still evolving; jmzReader's authors note the library 'is still evolving' and that they are 'committed to expand this library'.

## Evidence
