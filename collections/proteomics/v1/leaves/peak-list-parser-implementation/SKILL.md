---
name: peak-list-parser-implementation
description: Use when raw peak list files (MS2, PKL, MGF, DTA) or XML-based MS formats
  must be converted into structured spectrum records with precursor m/z, charge, and
  peak lists, especially for mzIdentML-driven workflows.
when_to_use_negative:
- Input is already a structured format like mzML or PRIDE XML whose spectra are accessible
  via an existing parser API rather than requiring a new custom parser
- The task is visualization or format conversion to MGF rather than implementing a
  parser — the jmzReader-gui already supports all library formats and can export them
  to mgf
- Spectra are embedded in mzIdentML files, which per the paper do not contain spectra
  data but only reference external peak list files
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: jmzReader
  role: 'Reference implementation: collection of Java APIs whose custom line-by-line
    text file parsing class and common parser interface the new parsers are modeled
    on'
  repo: http://code.google.com/p/jmzreader/
- name: Java
  role: Implementation language for the peak list parsers, following the jmzReader
    Java API design
- name: Maven
  role: Build/dependency manager used to pull jmzReader (uk.ac.ebi.pride.tools:jmzreader)
    and build the parser project
  repo: https://github.com/PRIDE-Utilities/jmzReader
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:56856bee0ed2bfbbd25ce10b3c71a4a7f7087250668ddb649d07747161019062
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzReader
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# peak-list-parser-implementation

## Summary

Implement memory-efficient line-by-line parsers for MS peak list formats (dta, ms2, mgf, pkl) and XML-based formats (mzXML, mzData, mzML, PRIDE XML) following the jmzReader design, emitting structured per-spectrum records with header metadata and m/z-intensity peak lists.

## When to use

Use when raw peak list files (MS2, PKL, MGF, DTA) or XML-based MS formats must be converted into structured spectrum records with precursor m/z, charge, and peak lists, especially for mzIdentML-driven workflows.

## When NOT to use

- Input is already a structured format like mzML or PRIDE XML whose spectra are accessible via an existing parser API rather than requiring a new custom parser
- The task is visualization or format conversion to MGF rather than implementing a parser — the jmzReader-gui already supports all library formats and can export them to mgf
- Spectra are embedded in mzIdentML files, which per the paper do not contain spectra data but only reference external peak list files

## Inputs

- MS2 peak list file (charge state, precursor m/z, retention time headers plus m/z-intensity peak lines)
- PKL peak list file (spectrum header lines with precursor m/z, charge, intensity, followed by peak lines)
- MGF, DTA, mzXML, mzData, mzML, or PRIDE XML mass spectrometry files
- Sample test files from the jmzReader repository test resources

## Outputs

- One structured record per spectrum containing header fields (charge, precursor m/z, retention time or precursor intensity as applicable) and an m/z-intensity peak list, e.g. serialized to ms2_spectra.json or pkl_spectra.json

## How to apply

Obtain a sample file for the target format (e.g. MS2 or PKL) from the jmzReader repository test resources as parser input. Implement a Java parser modeled on jmzReader's custom line-by-line text file parsing class, which lets all parsers handle arbitrarily large files in minimal memory. For MS2, parse the header structure to extract charge state, precursor m/z, and retention time when present, then read the associated m/z-intensity peak pairs; for PKL, read each spectrum's header line (precursor m/z, charge, precursor intensity) followed by its peak lines. Assemble one structured record per spectrum using a uniform schema, mirroring how all jmzReader parsers implement a common interface supporting the methods mzIdentML uses to reference external spectra, and serialize the records (e.g. to JSON). Validate that the output contains one record per input spectrum, each with the required header and non-empty peak list fields consistent with the source file.

## Related tools

- **jmzReader** (Reference implementation: collection of Java APIs whose custom line-by-line text file parsing class and common parser interface the new parsers are modeled on) — http://code.google.com/p/jmzreader/
- **Java** (Implementation language for the peak list parsers, following the jmzReader Java API design)
- **Maven** (Build/dependency manager used to pull jmzReader (uk.ac.ebi.pride.tools:jmzreader) and build the parser project) — https://github.com/PRIDE-Utilities/jmzReader

## Examples

```
mvn dependency:copy-dependencies && java -cp target/classes:target/dependency uk.ac.ebi.pride.tools.jmzreader.JmzReader --file sample.ms2
```

## Evaluation signals

- The parser runs on the jmzReader sample MS2/PKL test file without errors
- Output file exists and is valid JSON with one record per spectrum in the input file (record count equals spectrum count)
- Every emitted MS2 record carries charge state, precursor m/z, and (when present in the file) retention time fields
- Every emitted PKL record contains precursor m/z, charge, and a non-empty peak list consistent with the source file's content
- Peak m/z-intensity pairs in each record match the lines in the source peak list file

## Limitations

- jmzReader version numbers must be updated manually in the Maven dependency snippet; the README notes 'you need to change the version number to the latest version'
- mzIdentML files contain no spectra data — parsers must support the methods mzIdentML uses to reference external peak list files rather than reading spectra from the identification file
- The library is still evolving; the README states the authors are committed to expand it and add more classes, so interfaces may change

## Evidence
