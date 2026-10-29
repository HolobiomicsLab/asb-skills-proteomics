---
name: precursor-ion-metadata-extraction
description: Use when mzML files must be parsed in Java to recover per-spectrum precursor
  m/z and charge from spectrum metadata, alongside peak m/z and intensity arrays.
when_to_use_negative:
- Input is a plain peak list format (DTA, MS2, MGF, PKL) that carries no precursor
  metadata structure beyond what the peak list itself contains
- A different language stack is required — jmzReader is a collection of Java APIs
  and cannot be used directly outside the Java/Maven ecosystem
- 'Input is mzIdentML: mzIdentML files do not contain spectra data but refer to external
  peak list files, so no precursor or peak extraction from the mzIdentML file itself
  is possible'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: jmzReader
  role: Library whose mzML parser (built on jmzML) is used to iterate spectra and
    extract precursor ion metadata and peak lists
  repo: http://code.google.com/p/jmzreader/
- name: jmzML
  role: Already existing Java API upon which the jmzReader mzML parser is implemented
- name: Maven
  role: Build tool used to add the jmzReader parsers as dependencies via the pom file
    and EBI Nexus repositories
- name: Java
  role: Programming language in which the jmzReader APIs are written and the extraction
    is implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201100578
    title: 'jmzReader: A Java parser library to process and visualize multiple text
      and XML‐based mass spectrometry data formats'
  source_content_hash: sha256:5bc27ca2c04f96c5e5c13e0d1b2be5aa5327a3ed058c532fc7d21ace0906f17b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Maven
  - jmzML
  - jmzReader
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# precursor-ion-metadata-extraction

## Summary

Extract precursor ion metadata (precursor m/z and charge state where present) from mzML files by parsing spectra through the jmzReader mzML parser, which is built on the already existing jmzML Java API, and attaching precursor fields to per-spectrum records.

## When to use

Use when mzML files must be parsed in Java to recover per-spectrum precursor m/z and charge from spectrum metadata, alongside peak m/z and intensity arrays.

## When NOT to use

- Input is a plain peak list format (DTA, MS2, MGF, PKL) that carries no precursor metadata structure beyond what the peak list itself contains
- A different language stack is required — jmzReader is a collection of Java APIs and cannot be used directly outside the Java/Maven ecosystem
- Input is mzIdentML: mzIdentML files do not contain spectra data but refer to external peak list files, so no precursor or peak extraction from the mzIdentML file itself is possible

## Inputs

- mzML file (XML-based MS data format)
- Java project with Maven pom file
- uk.ac.ebi.pride.tools:jmzreader Maven dependency from EBI Nexus

## Outputs

- mzml_spectra.tsv — one row per spectrum with columns for index, MS level, precursor m/z, precursor charge (where present), and peak m/z and intensity arrays

## How to apply

Declare the uk.ac.ebi.pride.tools:jmzreader dependency (with the EBI Nexus pst-release and pst-snapshots repositories) in the Maven pom file, changing the version number to the latest version. Load the mzML file through the jmzReader mzML parser, which reuses the already existing jmzML API rather than an independent parser, and iterate over the spectra via the common jmzReader parser interface. For each spectrum record its index and MS level, then extract precursor ion information (precursor m/z and charge where present) from the mzML metadata. Extract the peak m/z and intensity arrays and attach them to the corresponding record. Write one row per spectrum with columns for index, MS level, precursor fields, and peak arrays. Rationale: all jmzReader parsers implement a common interface supporting the methods mzIdentML uses to reference external spectra, so only one interface must be supported.

## Related tools

- **jmzReader** (Library whose mzML parser (built on jmzML) is used to iterate spectra and extract precursor ion metadata and peak lists) — http://code.google.com/p/jmzreader/
- **jmzML** (Already existing Java API upon which the jmzReader mzML parser is implemented)
- **Maven** (Build tool used to add the jmzReader parsers as dependencies via the pom file and EBI Nexus repositories)
- **Java** (Programming language in which the jmzReader APIs are written and the extraction is implemented)

## Examples

```
<dependency><groupId>uk.ac.ebi.pride.tools</groupId><artifactId>jmzreader</artifactId><version>1.2.3</version></dependency> // then: MzMLFile file = new MzMLFile(new File("sample.mzML"));
```

## Evaluation signals

- mzml_spectra.tsv exists and is well-formed TSV with the required fields (index, MS level, precursor, peaks)
- Row count of mzml_spectra.tsv equals the number of spectra in the input mzML file
- Every spectrum row carries precursor m/z and charge fields where present in the mzML metadata, with MS level recorded per spectrum
- Peak m/z and intensity arrays extracted for every spectrum match the arrays in the source mzML records
- The common jmzReader interface is used so the same iteration code works across other supported formats (mzXML, mzData, MGF, etc.)

## Limitations

- The jmzReader version number must be manually changed to the latest version in the Maven pom; the documented snippet uses a placeholder
- Precursor charge is extracted only 'where present' — mzML spectra may lack precursor ion information
- The library is still evolving; the README notes a commitment to expand the library and add more classes

## Evidence
