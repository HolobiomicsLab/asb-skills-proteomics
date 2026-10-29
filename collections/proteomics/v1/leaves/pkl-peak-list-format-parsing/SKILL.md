---
name: pkl-peak-list-format-parsing
description: Use when the input is a PKL peak list file and structured per-spectrum
  records (precursor m/z, charge, peak list) are needed, e.g. to reconstruct or reimplement
  a PKL parsing component in Java.
when_to_use_negative:
- The input is already an mzML, mzXML, mzData, PRIDE XML, DTA, MS2, or MGF file —
  use the corresponding jmzReader parser for that format instead of the PKL parser.
- The goal is visualization or export to mgf rather than programmatic parsing — the
  jmzReader-gui already supports all library formats and can export them to mgf.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Provides the PKL parser among its collection of Java APIs for the most commonly
    used peak list formats; all parsers implement a common interface for use with
    mzIdentML
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Programming language used to implement the PKL parsing via jmzReader's Java
    APIs and its memory-efficient line-by-line text file parsing class
- name: Maven
  role: Build system used to add jmzReader (uk.ac.ebi.pride.tools:jmzreader) and its
    dependencies from the EBI Nexus repositories
  repo: https://github.com/PRIDE-Utilities/jmzReader/actions/workflows/maven.yml
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:57bd8841039220f8a9285ce40e40bc92a8b2b49c2c061442346496c2b6ca1b1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzReader
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# pkl-peak-list-format-parsing

## Summary

Parse PKL peak list files into structured spectrum records (precursor m/z, charge state, precursor intensity, and m/z–intensity peak pairs) using jmzReader's Java PKL parser, one of its common-interface parsers for DTA, MS2, MGF, PKL, mzXML, mzData, and mzML formats.

## When to use

Use when the input is a PKL peak list file and structured per-spectrum records (precursor m/z, charge, peak list) are needed, e.g. to reconstruct or reimplement a PKL parsing component in Java.

## When NOT to use

- The input is already an mzML, mzXML, mzData, PRIDE XML, DTA, MS2, or MGF file — use the corresponding jmzReader parser for that format instead of the PKL parser.
- The goal is visualization or export to mgf rather than programmatic parsing — the jmzReader-gui already supports all library formats and can export them to mgf.

## Inputs

- PKL peak list file (spectrum header lines with precursor m/z, charge, intensity followed by m/z–intensity peak lines)
- Maven project (pom.xml) for adding the jmzReader dependency
- Sample PKL test resource from PRIDE-Utilities/jmzReader

## Outputs

- Structured spectrum records combining precursor m/z, charge state, and the parsed peak list
- pkl_spectra output file with one record per spectrum

## How to apply

Add jmzReader to your Maven project (groupId uk.ac.ebi.pride.tools, artifactId jmzreader) from the EBI Nexus repositories (pst-release / pst-snapshots), updating to the latest version. Obtain a sample PKL file from the PRIDE-Utilities/jmzReader repository test resources. Parse each spectrum header line to extract precursor m/z, charge state, and precursor intensity, then read the following peak lines as m/z–intensity pairs. Rely on jmzReader's custom line-by-line text file parsing class, which handles arbitrarily large files in minimal memory, and assemble one structured record per spectrum under a uniform schema, mirroring the common interface all jmzReader peak list parsers implement for mzIdentML cross-referencing. Validate that every record carries precursor m/z, charge, and a non-empty peak list consistent with the source file.

## Related tools

- **jmzReader** (Provides the PKL parser among its collection of Java APIs for the most commonly used peak list formats; all parsers implement a common interface for use with mzIdentML) — http://code.google.com/p/jmzreader/
- **Java** (Programming language used to implement the PKL parsing via jmzReader's Java APIs and its memory-efficient line-by-line text file parsing class)
- **Maven** (Build system used to add jmzReader (uk.ac.ebi.pride.tools:jmzreader) and its dependencies from the EBI Nexus repositories) — https://github.com/PRIDE-Utilities/jmzReader/actions/workflows/maven.yml

## Examples

```
mvn archetype:generate … ; add <dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>latest</version></dependency> to pom.xml, then PklsFile pkl = new PklsFile(new File("sample.pkl"));
```

## Evaluation signals

- The parser runs end-to-end on the sample PKL file from the jmzReader test resources without errors.
- Every emitted spectrum record contains a precursor m/z, a charge state, and a non-empty peak list.
- The number of output records equals the number of spectrum headers in the source PKL file.
- Peak m/z–intensity pairs in each record match the peak lines in the source PKL file (diff against a manual parse of the sample).
- Records conform to a uniform schema consistent with jmzReader's common parser interface used across DTA, MS2, MGF, PKL, mzXML, mzData, and mzML parsers.

## Limitations

- jmzReader version number must be updated to the latest version in the Maven dependency snippet; the README gives only a placeholder 'version'.
- mzIdentML files do not contain spectra data but refer to external peak list files, so parsing alone does not resolve identifications; the common interface exists for cross-referencing external spectra.
- The library is still evolving — the README notes the authors 'are committed to expand this library and add more useful classes'.

## Evidence
