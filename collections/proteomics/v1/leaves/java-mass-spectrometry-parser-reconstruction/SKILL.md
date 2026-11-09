---
name: java-mass-spectrometry-parser-reconstruction
description: Use when a text-format peak list file (e.g. MGF with BEGIN IONS/END IONS
  blocks) must be converted into structured spectrum records with per-spectrum parameters
  and peak lists in a memory-efficient streaming design.
when_to_use_negative:
- Input is already an XML-based structured format (mzML, mzXML, mzData, PRIDE XML)
  that needs an XML API rather than line-by-line text parsing
- The input is an mzIdentML file, which contains no spectra but only references external
  peak list files
- A full-featured GUI viewer/exporter is needed rather than a programmatic parser
  — jmzReader-gui already visualizes peak list files and exports supported formats
  to mgf
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3368
tools:
- name: jmzReader
  role: Reference Java parser library whose custom line-by-line text-parsing design
    and common peak list interface are reconstructed by this skill
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language for the line-by-line parser producing structured spectrum
    records
- name: jmzReader-gui
  role: Bundled viewer demonstrating jmzReader interface usage and exporting all supported
    formats into mgf
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:d9447a0af68c8c070f678aeed9ab8fc1957f45695fe04c4b6e24e35fe9b9101c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# java-mass-spectrometry-parser-reconstruction

## Summary

Reconstructs an efficient line-by-line text parser for mass spectrometry peak list files (MGF, DTA, MS2, PKL) in Java, mirroring jmzReader's design so arbitrarily large files can be processed in minimal memory.

## When to use

Use when a text-format peak list file (e.g. MGF with BEGIN IONS/END IONS blocks) must be converted into structured spectrum records with per-spectrum parameters and peak lists in a memory-efficient streaming design.

## When NOT to use

- Input is already an XML-based structured format (mzML, mzXML, mzData, PRIDE XML) that needs an XML API rather than line-by-line text parsing
- The input is an mzIdentML file, which contains no spectra but only references external peak list files
- A full-featured GUI viewer/exporter is needed rather than a programmatic parser — jmzReader-gui already visualizes peak list files and exports supported formats to mgf

## Inputs

- Text-format MGF peak list file (BEGIN IONS / END IONS blocks with header parameters and peak lines)
- Other text peak list formats supported by jmzReader: DTA, MS2, PKL

## Outputs

- One structured JSON record per spectrum containing parameter fields (TITLE, PEPMASS, CHARGE, RTINSECONDS, etc.)
- Per-spectrum peak list as an array of (m/z, intensity) two-element numeric arrays

## How to apply

Load the target MGF file (the commonly used peak list format supported by jmzReader) and parse it line by line — the same custom-built class strategy jmzReader uses to handle arbitrary large files in minimal memory — splitting the text into BEGIN IONS / END IONS spectrum blocks. Within each block, extract PARAMETER-style header lines (TITLE, PEPMASS, CHARGE, RTINSECONDS, etc.) as key-value metadata, coercing PEPMASS and RTINSECONDS to numeric values, and parse the remaining peak lines into (m/z, intensity) pairs stored as two-element numeric arrays. Emit one structured JSON record per spectrum containing the parameter fields and peak list. The rationale is that mzIdentML files contain no spectra themselves but refer to external peak list files, so a common line-by-line interface over all peak list formats (DTA, MS2, MGF, PKL) lets one parser serve all downstream consumers. Validate by counting BEGIN IONS blocks and confirming record counts, required parameter fields, non-empty peak arrays, and fully numeric m/z and intensity entries.

## Related tools

- **jmzReader** (Reference Java parser library whose custom line-by-line text-parsing design and common peak list interface are reconstructed by this skill) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language for the line-by-line parser producing structured spectrum records)
- **jmzReader-gui** (Bundled viewer demonstrating jmzReader interface usage and exporting all supported formats into mgf)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>version</version><type>jar</type></dependency>
```

## Evaluation signals

- Number of emitted records in mgf_spectra equals the number of BEGIN IONS blocks in the input MGF file
- Every record contains the required parameter fields (TITLE, PEPMASS, CHARGE, RTINSECONDS) and a non-empty peak array
- All peak m/z and intensity entries parse as valid numbers after numeric coercion
- Memory footprint stays bounded (minimal memory) regardless of file size, consistent with the line-by-line streaming design
- Parsed PEPMASS and RTINSECONDS values are numeric, matching their coercions from header text

## Limitations

- jmzReader version number must be changed to the latest version when adding the Maven dependency; the documented snippet uses a placeholder
- The library is still evolving; the authors commit to expanding it and adding more useful classes
- mzIdentML files contain no spectra data — they only refer to external peak list files, so this parser handles the referenced files, not mzIdentML itself

## Evidence
