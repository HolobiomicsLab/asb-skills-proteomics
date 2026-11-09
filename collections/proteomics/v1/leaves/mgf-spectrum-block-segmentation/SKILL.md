---
name: mgf-spectrum-block-segmentation
description: Use when the input is a text-format MGF (Mascot Generic Format) peak
  list file that must be segmented into individual spectra with parameter metadata
  and (m/z, intensity) peak lists, e.g. for downstream mzIdentML-referenced spectrum
  lookups.
when_to_use_negative:
- The input is an XML-based mass spectrometry format (mzML, mzData, mzXML, PRIDE XML)
  rather than a text-format peak list — the BEGIN IONS / END IONS block segmentation
  does not apply
- The input is a single-spectrum DTA file with no BEGIN IONS / END IONS delimiters
  to segment
- The input is already a structured spectrum record collection (e.g. a prior mgf_spectra
  JSON output) — re-segmentation would be redundant
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: jmzReader
  role: Java API library whose MGF parser and custom line-by-line text parsing class
    define the efficient parsing mechanism this skill mirrors; also provides jmzReader-gui
    to export all supported file formats into the commonly used mgf format
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language of the jmzReader peak list parsing APIs referenced
    by this skill
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:ebdbd6362a0c240d49de98911d9e013581cb41e0672aaf555991c17378fff29d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - jmzReader
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# mgf-spectrum-block-segmentation

## Summary

Segment a text-format MGF peak list file into BEGIN IONS / END IONS spectrum blocks and extract per-spectrum parameter headers (TITLE, PEPMASS, CHARGE, RTINSECONDS) and peak lists into structured records. Mirrors the jmzReader library's line-by-line text parsing approach, which lets parsers handle arbitrarily large files in minimal memory.

## When to use

Use when the input is a text-format MGF (Mascot Generic Format) peak list file that must be segmented into individual spectra with parameter metadata and (m/z, intensity) peak lists, e.g. for downstream mzIdentML-referenced spectrum lookups.

## When NOT to use

- The input is an XML-based mass spectrometry format (mzML, mzData, mzXML, PRIDE XML) rather than a text-format peak list — the BEGIN IONS / END IONS block segmentation does not apply
- The input is a single-spectrum DTA file with no BEGIN IONS / END IONS delimiters to segment
- The input is already a structured spectrum record collection (e.g. a prior mgf_spectra JSON output) — re-segmentation would be redundant

## Inputs

- Text-format MGF (Mascot Generic Format) peak list file, e.g. the sample MGF file from the jmzReader repository test resources
- Optionally, other peak list files exported to the commonly used mgf format (e.g. via jmzReader-gui)

## Outputs

- One structured JSON record per spectrum in mgf_spectra, containing parameter fields (TITLE, PEPMASS, CHARGE, RTINSECONDS, etc.) and a non-empty array of (m/z, intensity) two-element numeric peak pairs

## How to apply

Load the MGF file and parse it line by line (the same custom-built line-by-line text parsing design jmzReader uses to handle arbitrarily large files in minimal memory). Split the stream into spectra at BEGIN IONS / END IONS delimiters. Within each block, extract PARAMETER-style header lines (TITLE, PEPMASS, CHARGE, RTINSECONDS, etc.) as key–value metadata, coercing PEPMASS and RTINSECONDS to numeric values. Parse the remaining lines into (m/z, intensity) pairs stored as two-element numeric arrays. Emit one structured JSON record per spectrum containing the parameter fields and the peak list, writing all records to mgf_spectra. Validate that the record count equals the number of BEGIN IONS blocks and that all peak values parse as valid numbers.

## Related tools

- **jmzReader** (Java API library whose MGF parser and custom line-by-line text parsing class define the efficient parsing mechanism this skill mirrors; also provides jmzReader-gui to export all supported file formats into the commonly used mgf format) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language of the jmzReader peak list parsing APIs referenced by this skill)

## Examples

```
python scripts/mgf_block_parser.py --input sample.mgf --output mgf_spectra.
```

## Evaluation signals

- The number of records in mgf_spectra equals the number of BEGIN IONS blocks in the input MGF file
- Every emitted record contains the required parameter fields (e.g. TITLE, PEPMASS, CHARGE) and a non-empty peak array
- All peak m/z and intensity entries parse as valid numbers and each peak is a two-element numeric array
- PEPMASS and RTINSECONDS values in each record are stored as numeric types, not raw strings
- Parsing proceeds line by line with bounded memory (no full-file slurp), consistent with the jmzReader design claim for arbitrarily large files

## Limitations

- The jmzReader version number must be set to the latest version when adding the parser as a Maven dependency; the documented snippet leaves it as a placeholder
- The MGF parser is designed for text-format peak list files; mzIdentML files do not contain spectra data but refer to external peak list files, so this skill only covers the peak list side
- The jmzReader library is still evolving and the maintainers are committed to expanding it, so parser behavior may change

## Evidence
