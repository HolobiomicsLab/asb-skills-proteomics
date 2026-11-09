---
name: mgf-parameter-block-extraction
description: Use when the input is a text-format mass spectrometry peak list file
  in MGF format (or a format jmzReader can export to MGF) and you need structured
  per-spectrum records with parameters like TITLE, PEPMASS, CHARGE, RTINSECONDS and
  peak arrays.
when_to_use_negative:
- Input spectra are already in a structured XML format such as mzML, mzXML, mzData,
  or PRIDE XML and you can use the corresponding jmzReader parser directly rather
  than converting through MGF
- The file is a binary MS format (e.g. raw instrument files) rather than a text peak
  list format — line-by-line MGF parsing does not apply
- You need chromatographic or acquisition-level metadata not representable in MGF
  spectrum blocks
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: Java parser library whose MGF parser and custom line-by-line text parsing
    class define the parsing mechanism this skill mirrors; usable as a Maven dependency
    (uk.ac.ebi.pride.tools:jmzreader)
  repo: http://code.google.com/p/jmzreader/
- name: jmzReader-gui
  role: Viewer tool that can export all jmzReader-supported file formats into MGF,
    providing a way to obtain MGF inputs for this skill
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language of jmzReader's peak list parsers; the library exposes
    Java APIs for efficient processing of peak list files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:97205efc41966081eae6dbbcd1838983b21944cdadc254d8f112a4528d5e43cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - jmzReader
  - jmzReader-gui
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mgf-parameter-block-extraction

## Summary

A skill for parsing MGF (Mascot Generic Format) peak list files line by line, splitting them into BEGIN IONS / END IONS spectrum blocks and extracting per-spectrum parameter key–value pairs plus (m/z, intensity) peak lists — mirroring the efficient text-parsing design of the jmzReader library.

## When to use

Use when the input is a text-format mass spectrometry peak list file in MGF format (or a format jmzReader can export to MGF) and you need structured per-spectrum records with parameters like TITLE, PEPMASS, CHARGE, RTINSECONDS and peak arrays.

## When NOT to use

- Input spectra are already in a structured XML format such as mzML, mzXML, mzData, or PRIDE XML and you can use the corresponding jmzReader parser directly rather than converting through MGF
- The file is a binary MS format (e.g. raw instrument files) rather than a text peak list format — line-by-line MGF parsing does not apply
- You need chromatographic or acquisition-level metadata not representable in MGF spectrum blocks

## Inputs

- MGF format peak list text file (e.g. a sample MGF file from the jmzReader repository test resources)
- Optionally, any file format supported by jmzReader (dta, ms2, mzData, mzXML, pkl, mzML, PRIDE XML) exported to MGF via jmzReader-gui

## Outputs

- One structured record per spectrum (e.g. JSON) containing parameter fields (TITLE, PEPMASS, CHARGE, RTINSECONDS, etc.) and a peak list of (m/z, intensity) two-element numeric arrays, written to mgf_spectra

## How to apply

Load the MGF file and scan it line by line (jmzReader's design principle: a custom class parses text files line by line so arbitrarily large files are handled in minimal memory), splitting on BEGIN IONS / END IONS delimiters to delimit spectrum blocks. Within each block, collect PARAMETER-style header lines as key–value metadata (e.g. TITLE, PEPMASS, CHARGE, RTINSECONDS), coercing PEPMASS and RTINSECONDS to numeric values. Parse the remaining lines in each block into (m/z, intensity) pairs stored as two-element numeric arrays. Emit one structured record (e.g. JSON) per spectrum, then validate: record count equals the number of BEGIN IONS blocks, every record carries the required parameter fields and a non-empty peak array, and all peak m/z and intensity entries parse as valid numbers. Rationale: jmzReader provides a dedicated MGF parser among its Java peak list parsers (dta, mgf, ms2, mzData, mzXML, pkl, mzML, PRIDE XML), all built on the same line-by-line parsing class.

## Related tools

- **jmzReader** (Java parser library whose MGF parser and custom line-by-line text parsing class define the parsing mechanism this skill mirrors; usable as a Maven dependency (uk.ac.ebi.pride.tools:jmzreader)) — http://code.google.com/p/jmzreader/
- **jmzReader-gui** (Viewer tool that can export all jmzReader-supported file formats into MGF, providing a way to obtain MGF inputs for this skill) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language of jmzReader's peak list parsers; the library exposes Java APIs for efficient processing of peak list files)

## Examples

```
import uk.ac.ebi.pride.tools.mgf_parser.MgfFile; MgfFile mgf = new MgfFile(new File("sample.mgf")); for (String title : mgf.getSpectraIds()) { PeakListSpectrum s = mgf.getSpectrumById(title); }
```

## Evaluation signals

- The number of records in mgf_spectra equals the number of BEGIN IONS blocks in the input MGF file
- Every emitted record contains the required parameter fields (e.g. TITLE, PEPMASS, CHARGE, RTINSECONDS) and a non-empty peak array
- All peak m/z and intensity entries parse as valid numbers (each stored as a two-element numeric array)
- PEPMASS and RTINSECONDS values in the output are numeric (successfully coerced), not raw strings
- For large files, memory usage stays bounded because parsing proceeds line by line rather than loading the whole file

## Limitations

- jmzReader version numbers in the documented Maven snippets are placeholders — you must change the version number to the latest version
- The library is still evolving; the README notes the team is committed to expand the library and add more useful classes
- MGF blocks only carry the parameters present in the file; absence of a field like RTINSECONDS means no numeric coercion is possible for that record

## Evidence
