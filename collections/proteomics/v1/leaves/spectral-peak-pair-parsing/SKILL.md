---
name: spectral-peak-pair-parsing
description: Use when input is a text-format peak list file such as MGF (with BEGIN
  IONS / END IONS blocks, TITLE/PEPMASS/CHARGE/RTINSECONDS headers and peak lines)
  and you need structured spectrum records with peak pairs for downstream search or
  QC.
when_to_use_negative:
- Input is already a structured format like mzML, mzXML, mzData or PRIDE XML where
  you should use the corresponding jmzReader XML-based parser rather than MGF line
  parsing
- You need a standalone viewer or export utility rather than programmatic parsing
  — the jmzReader-gui can visualize peak list files and export them to MGF instead
- mzIdentML-based software that only needs to reference external spectra through the
  common jmzReader parser interface rather than parse peak files directly
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Reference Java API whose custom line-by-line text parsing class defines the
    memory-efficient parsing mechanism reproduced by this skill
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Programming language in which jmzReader's peak list parsers and the line-by-line
    parsing mechanism are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:a0505a66e6d63852b70eedbaf9cf7d9f14c0f6e6c871827da2fd1868ff407aa4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - jmzReader
  edam_operation: http://edamontology.org/operation_0226
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

# spectral-peak-pair-parsing

## Summary

Parse text-format mass spectrometry peak list files (MGF, DTA, MS2, PKL) into structured spectrum records with per-spectrum parameters and (m/z, intensity) peak pairs, following jmzReader's efficient line-by-line parsing design so arbitrarily large files can be processed in minimal memory.

## When to use

Use when input is a text-format peak list file such as MGF (with BEGIN IONS / END IONS blocks, TITLE/PEPMASS/CHARGE/RTINSECONDS headers and peak lines) and you need structured spectrum records with peak pairs for downstream search or QC.

## When NOT to use

- Input is already a structured format like mzML, mzXML, mzData or PRIDE XML where you should use the corresponding jmzReader XML-based parser rather than MGF line parsing
- You need a standalone viewer or export utility rather than programmatic parsing — the jmzReader-gui can visualize peak list files and export them to MGF instead
- mzIdentML-based software that only needs to reference external spectra through the common jmzReader parser interface rather than parse peak files directly

## Inputs

- Text-format mass spectrometry peak list file, e.g. MGF with BEGIN IONS / END IONS blocks (jmzReader also supports dta, ms2, pkl and XML-based formats)
- Sample MGF file such as the test resources shipped with the jmzReader repository

## Outputs

- One structured record per spectrum containing parameter fields (TITLE, PEPMASS, CHARGE, RTINSECONDS) and a list of (m/z, intensity) pairs, e.g. JSON records written to mgf_spectra

## How to apply

Load the MGF file and split it into BEGIN IONS / END IONS spectrum blocks, mirroring jmzReader's approach of parsing text files line by line with a custom-built class. Within each block, extract PARAMETER-style header lines (TITLE, PEPMASS, CHARGE, RTINSECONDS, etc.) as key-value metadata, coercing PEPMASS and RTINSECONDS to numeric values. Parse the remaining peak lines into (m/z, intensity) pairs stored as two-element numeric arrays. Emit one structured record per spectrum containing the parameter fields and the peak list; the line-by-line design is the rationale for handling arbitrary large files in minimal memory, so avoid loading the whole file into memory at once.

## Related tools

- **jmzReader** (Reference Java API whose custom line-by-line text parsing class defines the memory-efficient parsing mechanism reproduced by this skill) — http://code.google.com/p/jmzreader/
- **Java** (Programming language in which jmzReader's peak list parsers and the line-by-line parsing mechanism are implemented)

## Examples

```
grep -c 'BEGIN IONS' sample.mgf; python parse_mgf.py --input sample.mgf --output mgf_spectra.
```

## Evaluation signals

- Number of emitted records in mgf_spectra equals the number of BEGIN IONS blocks in the input MGF file
- Every spectrum record contains the required parameter fields (TITLE, PEPMASS, CHARGE, RTINSECONDS) and a non-empty peak array
- All peak m/z and intensity entries parse as valid numbers
- Memory footprint stays bounded on very large MGF files, consistent with jmzReader's claim that all parsers can handle arbitrary large files in minimal memory

## Limitations

- The jmzReader version number is a placeholder in the documentation; you need to change it to the latest version when adding the Maven dependency
- The library is still evolving; the README states the developers are committed to expand the library and add more useful classes
- mgf is described as the commonly used export target, but native MGF content (headers beyond the standard parameters) may require custom handling since mzIdentML-oriented parsers expose a common interface rather than raw parameter access

## Evidence
